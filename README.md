# libbolt-go
Go Bindings for libbolt API

This package provides cgo bindings to [libbolt](https://github.com/boltlabs-inc/libbolt)
using its [C API](https://github.com/boltlabs-inc/libbolt/blob/master/include/libbolt.h).

### Install

First, you will need to [install Rust](https://www.rust-lang.org/downloads.html) (you'll
need Rust 1.30+) and have Go compiler installed. To run tests for `libbolt-go`, you'll first compile the libbolt release and then show the Go compiler where to find it. Here are the steps:

	git clone https://github.com/boltlabs-inc/libbolt.git
	cargo +nightly build --release --manifest-path ./libbolt/Cargo.toml
	export CGO_LDFLAGS="-L$(pwd)/libbolt/target/release"
	export LD_LIBRARY_PATH="$(pwd)/libbolt/target/release"
	
	go get github.com/boltlabs-inc/libbolt-go
	go test github.com/boltlabs-inc/libbolt-go

### Example usage

To import the library, do the following:

	import (
		"fmt"
		"github.com/boltlabs-inc/libbolt-go"
	)
	
To create new channel state:

	channelState, _ := libbolt.BidirectionalChannelSetup("New zkChannel", false)
	fmt.Println("channelState := ", channelState)
	
To initiate merchant state:
	
	channelToken, merchState, channelState, err := libbolt.BidirectionalInitMerchant(channelState, "Bob")
		
To initiate customer state:
	
	b0Cust := 1000
	b0Merch := 0
	channelToken, custState, err := libbolt.BidirectionalInitCustomer(channelToken, b0Cust, b0Merch, "Alice")
	
...

TODO: add channel establish and pay protocol usage

TODO: include channel closing