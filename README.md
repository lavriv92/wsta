# wsta

The WebSocket Transfer Agent

`wsta` is a cli tool written in rust for interfacing with WebSockets. `wsta` has
the simple philosophy of getting out of your way and letting you work your UNIX
magic on the WebSocket traffic directly. The way it does this is to be as
pipe-friendly as possible, letting you chain it into complex pipelines or bash
scripts as you see fit.

## Cool things you can do

Since `wsta` is really pipe-friendly, you can easily work with your output in
a way that suits you. If you have a websocket-service that returns JSON, you
might want to have your data printed in a nice, readable format.
[jq](https://stedolan.github.io/jq/) is perfect for that.

```bash
$ wsta ws://echo.websocket.org '{"test": "what?"}' | jq .
Connected to ws://echo.websocket.org
{
  "test": "what?"
}
```

If you're debugging some nasty problem with your stream, you are probably only
interested in frames related to your problem. Good news, `grep` is here to save
the day!

```bash
$ while true; do echo  $(( RANDOM %= 200 )); sleep 0.2; done | wsta ws://echo.websocket.org | grep '147'
147
147
147
147
147
147
```

Use `wsta` to monitor your websocket uptime. Use the `--ping` option to keep
the connection alive, and check the exit code for issues. You could also
potentially send the last few messages with POST data.

```bash
#!/bin/bash

while true; do

  # Start persistent connection, pinging evey 10 seconds to stay alive
  wsta --ping 10 ws://echo.websocket.org

  if [ $? -gt 0 ]; then
    curl -X POST https://SOUNDTHEALARM.yourcompany.com
  fi

  sleep 30
done
```

## Installation

### Requirements

Currently the only requirement to run `wsta` is openssl. If you get an error
about a missing `ssllib.so` or similar, search your package manager for openssl
or similar to add this library.

### Fedora
You run Fedora? You're in luck! I've set up a download page here that you can
get `wsta` from

https://software.opensuse.org/download.html?project=home%3Aesphen&package=wsta

I'm working on getting more distributions into the Open Build Service pipeline,
which is what creates the releases on that page. For now, you need Fedora to use
that page. If you don't use Fedora, have a look below at other options.

### Other 64-bit Linux distributions
I only have 64-bit Linux machines available to create binaries with, but if you
have a 64-bit Linux machine, I do provide the binary in every release here on
GitHub. Have a look at the most recent release, and place the attached binary
into your `$PATH`.

https://github.com/esphen/wsta/releases

### Compile it yourself

DON'T PANIC. It's really easy to compile and install `wsta` yourself! Rust
provides solid tools like `cargo` for automating the compilation. If you compile
`wsta` yourself, it should run on all of
[rust's supported platforms](https://doc.rust-lang.org/book/getting-started.html#platform-support).
I have only tested Linux, however, so YMMV.

    # Install the rust language and tools
    curl -sSf https://static.rust-lang.org/rustup.sh | sh -s -- --channel=beta

    # Install required libraries, using your distribution's package manager
    dnf install -y gcc openssl-devel

    # Clone the git repository
    mkdir -p ~/workspace
    cd ~/workspace
    git clone https://github.com/esphen/wsta.git
    cd wsta

    # Finally: compile and install it to your system
    sudo cargo install --root /usr/local

    # This will create a file called .crates.toml. You can delete it if you want
    sudo rm /usr/local/.crates.toml

## Development setup

[Install the rust language and
tools](https://doc.rust-lang.org/book/getting-started.html#installing-rust).
We use some cargo environment variables that are beta features (cargo 0.10).

    curl -sSf https://static.rust-lang.org/rustup.sh | sh -s -- --channel=beta

Run the program

    cargo run -- -vvv -I -e ws://echo.websocket.org

In order to generate the man page, `groff` is needed

    make man

