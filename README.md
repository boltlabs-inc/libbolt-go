# libbolt-go
Go Bindings for libbolt API

This package provides cgo bindings to [libbolt](https://github.com/boltlabs-inc/libbolt)
using its [C API](https://github.com/boltlabs-inc/libbolt/blob/master/include/libbolt.h).

### Install

First, you will need to [install Rust](https://www.rust-lang.org/downloads.html) (you'll
need Rust 1.30+) and have Go compiler installed. To run tests for `libbolt-go`, you'll first compile the libbolt release and then show the Go compiler where to find it. Here are the steps for building the `libbolt` lib:

	git clone https://github.com/boltlabs-inc/libbolt.git
	cargo +nightly build --release --manifest-path ./libbolt/Cargo.toml
	export CGO_LDFLAGS="-L$(pwd)/libbolt/target/release"
	export LD_LIBRARY_PATH="$(pwd)/libbolt/target/release"

To import the package in a different library:
	
	go get github.com/boltlabs-inc/libbolt-go
	go test github.com/boltlabs-inc/libbolt-go

To test locally, do the following:

	cd libbolt-go
	go get github.com/stretchr/testify/assert
	go test -v libbolt.go libbolt_test.go
	
### Example usage

## Channel setup

To import the library, do the following:

	import (
		"fmt"
		"github.com/boltlabs-inc/libbolt-go"
	)
	
## Channel init

To create new channel state:

	channelState, _ := libbolt.BidirectionalChannelSetup("New zkChannel", false)
	fmt.Println("channelState := ", channelState)
	
To initiate merchant state:
	
	channelToken, merchState, channelState, err := libbolt.BidirectionalInitMerchant(channelState, "Bob")
		
To initiate customer state:
	
	b0Cust := 1000
	b0Merch := 0
	channelToken, custState, err := libbolt.BidirectionalInitCustomer(channelToken, b0Cust, b0Merch, "Alice")
	

## Channel establish protocol

Customer generates the initial commitment proof for the first wallet commitment

	channelToken, custState, com, comProof, err := libbolt.BidirectionalEstablishCustomerGenerateProof(channelToken, custState)

Merchant verifies the commitment proof and generates an initial close token for the channel

	closeToken, err := libbolt.BidirectionalEstablishMerchantIssueCloseToken(channelState, com, comProof, custState.PkC, b0Cust, b0Merch, merchState)

Customer verifies the close token and can proceed with broadcasting escrow transaction if valid

	isTokenValid, channelState, custState, err := libbolt.BidirectionalVerifyCloseToken(channelState, custState, closeToken)

Merchant generates the pay token after confirming the escrow transaction and merchant close transaction

	payToken, err := libbolt.BidirectionalEstablishMerchantIssuePayToken(channelState, com, merchState)

Customer verifies the pay token and establishes the channel if token is valid

	isChannelEstablished, channelState, custState, err := libbolt.BidirectionalEstablishCustomerFinal(channelState, custState, payToken)

## Channel pay protocol

Phase 1 - Customer generates a payment proof and new customer state
	
	amount := 200
	payment, newCustState, err := libbolt.BidirectionalPayGeneratePaymentProof(channelState, custState, amount)

Phase 1 - Merchant verifies the payment proof and generates a close token

	closeToken, merchState, err = libbolt.BidirectionalPayVerifyPaymentProof(channelState, payment, merchState)

Phase 2 - Customer verifies the close token, updates the customer state and generates a revoke token for previous state of channel

	revokeToken, custState, err := libbolt.BidirectionalPayGenerateRevokeToken(channelState, custState, newCustState, closeToken)

Phase 2 - Merchant verifies the revoke token and generates the pay token for next run of pay protocol

	payToken, merchState, err = libbolt.BidirectionalPayVerifyRevokeToken(revokeToken, merchState)

Customer verifies the pay token and updates internal state with it if valid

	custState, isTokenValid, err := libbolt.BidirectionalPayVerifyPaymentToken(channelState, custState, payToken)

## Channel closing

To initiate a customer close, the customer first generates a close message as follows (then forms close tx):

	custClose, err := libbolt.BidirectionalCustomerClose(channelState, custState)
	# form cust close tx here
	
To check whether customer broadcasted an old state, the merchant does the following (with on-chain close message from customer during dispute period): 

	wpk, merchClose, err, _ := libbolt.BidirectionalMerchantClose(channelState, channelToken, "<pk-address>", custClose, merchState)
	# form merch close tx here

**TODO**: show merchant-initiated close
