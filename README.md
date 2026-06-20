# **roblox-rs**

```roblox-rs``` is a transpiler and library that allows writing Roblox scripts in Rust. It is designed to be a full language replacement for Lua/Luau similar to [*roblox-ts*](https://github.com/roblox-ts/roblox-ts).

## ⚠️ *Warning* ⚠️ 
***roblox-rs*** is currently a proof of concept and is in early development.

## Design Goals
* **Strict, but fair**: A balance between Rust's hard types and Lua's loose ownership model
* **Familiar to scripters**: A balance between Roblox's scripting flow and Rust's syntax
* **A better way to script**: Includes support for much of the Rust std library and an assortment of QoL features to make scripting more powerful and safer than Lua
* **Non-invasive**: Lightweight and easy to set up

## What It's Not
* A replacement for Roblox Studio
    * ***roblox-rs*** works *alongside* Roblox Studio
* A way to run Rust directly in Roblox
    * ***roblox-ts*** turns your code into valid Lua that can be run by Roblox

## Usage examples

**Client-Server Communication**
```Rust
use roblox_rs::*;

static msg_event: RemoteEvent<String> = wait_for("game.ReplicatedStorage.MsgEvent");

#[script(parent = "game.ServerScriptService")]
fn Server() {
    println!("Hello from server!");
    
    let _ = msg_event.on_server_event().connect(|_conn, (player, msg)| {
        println!("Received message `{}` from {}", msg, player.name);
    });
}

#[localscript(parent = "game.StarterPlayer.StarterPlayerScripts")]
fn Client() {
    println!("Hello from client!");

    msg_event.fire_server("This is a message".to_owned());
}
```

**Game Round Functionality**
```Rust
use roblox_rs::*;

const REQUIRED_PLAYERS: u32 = 6;

#[script(parent = "game.ServerScriptService")]
fn RoundHandler() {
    fn log_players(msg: impl AsRef<str>, num_players: u32) {
        println!("{} ({}/6)", msg, num_players);
    }

    fn start_round() {
        log_players("Starting round", num_players);
        // ...
    }

    let mut num_players = 0u32;
    let added_conn = PLAYERS.player_added().connect(|_conn, _player| {
        log_players("Waiting for players...", num_players);
        num_players += 1;
    });
    let removing_conn = PLAYERS.player_removing().connect(|_conn, _player| {
        log_players("Waiting for players...", num_players);
        num_players -= 1
    });

    for player in PLAYERS.get_players() {
        num_players += 1;
    }

    loop {
        task::wait(1);
        if num_players >= REQUIRED_PLAYERS {
            added_conn.disconnect();
            removing_conn.disconnect();

            break;
        }
    }

    start_round();
    // ...
}
```
