# A long-running Tokio service

Prefer **construct → wire → start → join** over `new()` that silently spawns a worker and discards its handle. Before startup, the composition point can obtain subscriptions and connect services without racing them. After startup, it owns a handle that proves when the worker has finished. Sending a shutdown notification alone proves no such thing.

The template below is a complete `src/main.rs`. It uses Tokio 1 with the `macros`, `rt`, and `sync` features. `HighWaterMark` tracks the largest sample it has received. Subscribers need only the latest high-water mark, not every intermediate value. Input samples, in contrast, must not be coalesced: doing so could lose the maximum. The two channel choices express those different contracts.

```rust
use tokio::sync::mpsc;
use tokio::sync::oneshot;
use tokio::sync::watch;

struct HighWaterMark {
    sample_receiver: mpsc::Receiver<u64>,
    shutdown_receiver: oneshot::Receiver<()>,
    highest_sample: u64,
    highest_sample_sender: watch::Sender<u64>,
}

impl HighWaterMark {
    /// Start with a known sample so a subscriber has a value before the worker runs.
    fn new(initial_sample: u64, sample_receiver: mpsc::Receiver<u64>, shutdown_receiver: oneshot::Receiver<()>) -> Self {
        let (highest_sample_sender, _) = watch::channel(initial_sample);
        Self { sample_receiver, shutdown_receiver, highest_sample: initial_sample, highest_sample_sender }
    }

    /// Subscribe before starting the service. Updates may coalesce; the newest value never decreases.
    fn subscribe(&self) -> watch::Receiver<u64> {
        self.highest_sample_sender.subscribe()
    }

    async fn task(mut self) -> Self {
        loop {
            tokio::select! {
                _ = &mut self.shutdown_receiver => break,
                sample = self.sample_receiver.recv() => {
                    let Some(sample) = sample else { break };
                    if sample > self.highest_sample {
                        self.highest_sample = sample;
                        self.highest_sample_sender.send_replace(sample);
                    }
                }
            }
        }
        self
    }

    /// Transfer ownership to the worker. Joining recovers the final service, not a fresh worker.
    fn into_task(self) -> tokio::task::JoinHandle<Self> {
        tokio::spawn(self.task())
    }
}

#[tokio::main(flavor = "current_thread")]
async fn main() {
    let (sample_sender, sample_receiver) = mpsc::channel(8);
    let (shutdown_sender, shutdown_receiver) = oneshot::channel();
    let service = HighWaterMark::new(3, sample_receiver, shutdown_receiver);
    let mut highest_samples = service.subscribe();
    assert_eq!(*highest_samples.borrow_and_update(), 3);
    let service_task = service.into_task();

    sample_sender.send(11).await.expect("the service still owns its sample receiver");
    highest_samples.changed().await.expect("the service still owns its publisher");
    assert_eq!(*highest_samples.borrow_and_update(), 11);

    shutdown_sender.send(()).expect("the worker is running and its sample input is open");
    let service = service_task.await.expect("the worker must not panic or be cancelled");
    assert_eq!(service.highest_sample, 11);
}
```

The example uses private items because it fits in a single binary. In a reusable module, expose the service type, constructor, `into_task`, and any client factory to the scope that composes the service. Keep `task` and state private. The example intentionally exposes a Tokio watch receiver: its consumers are meant to have watch semantics. If consumers need a domain contract instead, return a domain handle.

## What the owner promises

- **Construction is separate from spawning, not necessarily from I/O.** A constructor may need initial data before it can supply a useful client. If it cannot establish a required invariant, return an appropriate error. Don't substitute a default merely to keep `new` infallible.
- **Acquire handles only where there is a client operation.** Don't add `client()` to every service for symmetry. Don't promise that a handle is cloneable unless its contract supports independent users.
- **Specify what ends the task.** Here, an explicit shutdown signal, loss of its sender, or end of the input stream ends the worker. Shutdown need not drain pending samples. If draining is required, stop accepting new work, finish the accepted work, then return. Document which guarantee the caller gets.
- **Use the right cleanup boundary.** Complete required asynchronous cleanup before returning from `task`. Use `Drop` for destruction obligations that can finish synchronously. Regaining `Self` does not mean the consumed channels or other resources are reusable.
- **Child work is not another top-level service by default.** A private session can expose an async method returning a result for its parent to supervise. Track children when their outcomes or completion matter. Detach bounded work only when abandoning it at shutdown is correct. Giving the work a timeout does not settle that question.

## Degrade without concealing failure

Consider a listener receiving configuration updates and serving independent client sessions. A malformed request can terminate one session while the listener continues accepting others. An unavailable listen address can leave the listener inactive while the service still receives configuration and shutdown signals. If it retries, the owner must know why retry is safe and when it should stop. Don't add a blanket retry loop.

Distinguish a failed operation from an instruction to disable a feature. If a reload cannot be read, retaining the last accepted configuration may be correct. If a valid configuration explicitly disables the listener, retaining it would be wrong. Decide what an invalid replacement means for this service; last-known-good is not a universal policy. Make degradation observable without reporting success for work the service did not perform.

A worker panic has a different meaning from an ordinary client failure. Don't turn a violated safety invariant into “bad request” or blindly restart a service whose state you no longer trust. Handle join outcomes deliberately at the composition point.
