# Modular Light Indexer Set Up Guide

### 0. Get Your Gas Coupon

A gas coupon is essential for supporting your transactions on Nubit’s Data Availability (DA) layer, unlocked at 2000 points. Visit [points.nubit.org](points.nubit.org) to connect your wallet and invite friends to unlock your gas coupon.


### 1. Requirements

Ensure your device meets the following minimal specs for a smooth installation:

- CPU: Single Core
- Memory: 512 MB
- Disk: 30 GB
- Bandwidth: 100 KB/s upload/download


### 2. Clone Github Repository

Navigate to [Modular Indexer on GitHub](https://github.com/RiemaLabs/modular-indexer-light) and clone the repository to your local machine.


### 3. Install Dependencies

The Light Indexer is developed in Golang. Install the necessary dependencies with Go version 1.22.0 or higher, available on the [Golang download page](https://go.dev/dl/). Install all required packages by executing:

```bash
go mod tidy
```


### 4. Configuration Instructions

Set up your Light Indexer by starting with the example configuration file:
```bash
cp config.example.json config.json
```

Add your gas coupon and related information to configure your light indexer. For detailed configuration instructions, refer to the [GitHub readme](https://github.com/RiemaLabs/modular-indexer-light?tab=readme-ov-file#detailed-configuration-instructions).


### 5. Running the Program

Execute the following commands to initiate API services and upload checkpoints to the DA:

```bash
go build
./modular-indexer-light
```






