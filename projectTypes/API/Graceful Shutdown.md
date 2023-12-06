
- Shutdown signals — what they are, how to send them, and how to listen for them in your API application.
    
- How to use these signals to trigger a graceful shutdown of the HTTP server using Go’s [`Shutdown()`](https://golang.org/pkg/net/http/#Server.Shutdown) method.

## Shutdown Signals

<table>
<thead>
<tr>
<th>Signal</th>
<th>Description</th>
<th>Keyboard shortcut</th>
<th>Catchable</th>
</tr>
</thead>

<tbody>
<tr>
<td><code>SIGINT</code></td>
<td>Interrupt from keyboard</td>
<td><code>Ctrl+C</code></td>
<td>Yes</td>
</tr>

<tr>
<td><code>SIGQUIT</code></td>
<td>Quit from keyboard</td>
<td><code>Ctrl+\</code></td>
<td>Yes</td>
</tr>

<tr>
<td><code>SIGKILL</code></td>
<td>Kill process (terminate immediately)</td>
<td>-</td>
<td>No</td>
</tr>

<tr>
<td><code>SIGTERM</code></td>
<td>Terminate process in orderly manner</td>
<td>-</td>
<td>Yes</td>
</tr>
</tbody>
</table>

## Intercepting Shutdown Signals
To catch the signals, we’ll need to spin up a background goroutine which runs for the lifetime of our application. In this background goroutine, we can use the [`signal.Notify()`](https://godoc.org/os/signal#Notify) function to listen for specific signals and relay them to a channel for further processing.

But the important thing is that it demonstrates the pattern of how to catch specific signals and handle them in your code.

One thing I’d like to quickly emphasize about this: our `quit` channel is a [buffered channel](https://gobyexample.com/channel-buffering) with size 1.

We need to use a buffered channel here because `signal.Notify()` [does not wait](https://github.com/golang/go/blob/bc7e4d9257693413d57ad467814ab71f1585a155/src/os/signal/signal.go#L243) for a receiver to be available when sending a signal to the `quit` channel. If we had used a regular (non-buffered) channel here instead, a signal could be ‘missed’ if our `quit` channel is not ready to receive at the exact moment that the signal is sent. By using a buffered channel, we avoid this problem and ensure that we never miss a signal.

## Executing the Shutdown

Specifically, after receiving one of these signals we will call the [`Shutdown()`](https://golang.org/pkg/net/http/#Server.Shutdown) method on our HTTP server. The official documentation describes this as follows:
> Shutdown gracefully shuts down the server without interrupting any active connections. Shutdown works by first closing all open listeners, then closing all idle connections, and then waiting indefinitely for connections to return to idle and then shut down.

The pattern to implement this in practice is difficult to describe with words, so let’s jump into the code and talk through the details as we go along.

what it’s doing can be summarized very simply: _when we receive a `SIGINT` or `SIGTERM` signal, we instruct our server to stop accepting any new HTTP requests, and give any in-flight requests a ‘grace period’ of 30 seconds to complete before the application is terminated_.

It’s important to be aware that the `Shutdown()` method does not wait for any background tasks to complete, nor does it close hijacked long-lived connections like WebSockets. Instead, you will need to implement your own logic to coordinate a graceful shutdown of these things. We’ll look at some techniques for doing this later in the book.