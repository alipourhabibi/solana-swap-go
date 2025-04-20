
# Solana Swap Go

Go reimplementation of [solana-swap-python](https://github.com/YZYLAB/solana-swap-python.git)

Easiest way to add Solana based swaps to your project.
Uses the Solana Swap api from [https://docs.solanatracker.io](https://docs.solanatracker.io)

## Now supporting
- Raydium
- Raydium CPMM
- Pump.fun
- Moonshot
- Orca
- Jupiter (Private Self Hosted API)

## Installation

```bash
go get github.com/alipourhabibi/solana-swap-go
```

<!-- ## Demo

Swap API is used live on:
https://www.solanatracker.io

*Add your site here* -->

## Example Usage

```go
package main

import (
	"fmt"
	"log"
	"time"

	"github.com/gagliardetto/solana-go"
	"github.com/gagliardetto/solana-go/rpc"
	solanatracker "github.com/alipourhabibi/solana-swap-go"
)

func main() {
	startTime := time.Now()
	privateKey := "YOUR_SECRET_KEY" // replace with your base58 private key

	// Create a keypair from the secret key
	keypair, err := solana.PrivateKeyFromBase58(privateKey)
	if err != nil {
		log.Fatalf("Error creating keypair: %v", err)
	}

	rpcUrl := "https://solana-rpc.publicnode.com"

	// Initialize a new Solana tracker with the keypair and RPC endpoint
	tracker := solanatracker.NewSolanaTracker(keypair, rpcUrl)

	priorityFee := 0.00005 // priorityFee requires a pointer, thus we store it in a variable

	// Get the swap instructions for the specified tokens, amounts, and other parameters
	swapResponse, err := tracker.GetSwapInstructions(
		"So11111111111111111111111111111111111111112",
		"4k3Dyjzvzp8eMZWUXbBCjEvwSkkk59S5iCNLY3QrkX6R",
		0.0001,                       // Amount to swap
		30,                           // Slippage
		keypair.PublicKey().String(), // Payer public key
		&priorityFee,                 // Priority fee (Recommended while network is congested)
		false,
	)
	if err != nil {
		// Log and exit if there's an error getting the swap instructions
		log.Fatalf("Error getting swap instructions: %v", err)
	}

	maxRetries := uint(5) // maxRetries requires a pointer, thus we store it in a variable

	// Define the options for the swap transaction
	options := solanatracker.SwapOptions{
		SendOptions: rpc.TransactionOpts{
			SkipPreflight: true,
			MaxRetries:    &maxRetries,
		},
		ConfirmationRetries:        50,
		ConfirmationRetryTimeout:   1000 * time.Millisecond,
		LastValidBlockHeightBuffer: 200,
		Commitment:                 rpc.CommitmentProcessed,
		ResendInterval:             1500 * time.Millisecond,
		ConfirmationCheckInterval:  100 * time.Millisecond,
		SkipConfirmationCheck:      false,
	}

	// Perform the swap transaction with the specified options
	sendTime := time.Now()
	txid, err := tracker.PerformSwap(swapResponse, options)
	endTime := time.Now()
	elapsedTime := endTime.Sub(startTime).Seconds()
	if err != nil {
		fmt.Printf("Swap failed: %v\n", err)
		fmt.Printf("Time elapsed before failure: %.2f seconds\n", elapsedTime)
		// Add retries or additional error handling as needed
	} else {
		fmt.Printf("Transaction ID: %s\n", txid)
		fmt.Printf("Transaction URL: https://solscan.io/tx/%s\n", txid)
		fmt.Printf("Swap completed in %.2f seconds\n", elapsedTime)
		fmt.Printf("Transaction finished in %.2f seconds\n", endTime.Sub(sendTime).Seconds())
	}
}
```


## FAQ

#### Why should I use this API?

SolanaTracker retrieves all raydium tokens the second they are available, so you can perform fast snipes.
They also provide their own hosted Jupiter Swap API with no rate limits and faster market updates.

#### Is there a fee for using this API?

SolanaTracker charges a 0.5% fee on each successful transaction.

*From the original README:*

    Using this for a public bot or site with a high processing volume? 
    Contact us via Discord or email (solanatracker@yzylab.com) and get the fee reduced to 0.1% (only if accepted.)