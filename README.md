# Noir React Native starter

## Description

This is a simple React Native app showcasing how to use Noir in a mobile app (both for iOS and Android) to generate and verify proofs directly on mobile phones.

## Mobile proving

### iOS

The app integrates with the [Swoir library](https://github.com/Swoir/Swoir) to generate proofs with Noir on iOS. The library is written in Swift and is available as a Swift Package.

### Android

The app integrates some Kotlin code following a similar logic to Swoir, by taking the same type of inputs and the circuit manifest to generate proofs with Noir on Android. This part of the code will be exported soon in a separate library to simplify reusability.

## General setup

If you are unfamiliar with React Native, you can follow the [official guide](https://reactnative.dev/docs/environment-setup) to set up your environment.

For the rest follow the steps below:

1. Clone the repository
2. Run `npm install` to install the dependencies
3. [Optional] You can download the SRS to package it with the app binary and avoid fetching it from the server every time you generate a proof. Please refer to the section below on SRS download strategies.

## Setup on iOS

1. Run `npx pod-install` to install the pods for the iOS project
2. Open the project in Xcode
3. Make sure you see the `Swoir`, `SwoirCore` and `Swoirenberg` libraries in the `Package Dependencies` (if not please open an issue)
4. Make sure you have a valid provisioning profile set up for the app in `Signing & Capabilities`
5. Build & Run the app on your device

## Setup on Android

1. Make sure to define the environment variables `ANDROID_HOME`, `NDK_VERSION` and `HOST_TAG`, they will help the build process to find Android NDK necessary to compile the native code. Example on MacOS:

```bash
export ANDROID_HOME=$HOME/Library/Android/sdk
export NDK_VERSION=26.3.11579264
export HOST_TAG=darwin-x86_64
```

2. Connect your Android device and check it is connected by running `npm run android-devices`. It should display the connected device as `device` in the list of devices attached.
3. Run `npm run android` to build and run the app on your device

**Note**: If you want to do a clean build, you can run `./scripts/clean-android.sh` before running `npm run android`

## SRS download strategies

The Structured Reference String (or SRS) contains the necessary data from the Universal Trusted Setup of Aztec to generate proofs with Noir (or more precisely the Barretenberg backend). So you will need it in the app. Here's how to go about it:

### You have 5 minutes and already know the circuits you want to use

Then you should pre-download the SRS and package it with the app binary. You can do so by running the script `./scripts/download-srs.sh` that will help you do that in a single command. First, you will need to identify which of your circuits has the highest gate count, that is which one of them returns the biggest `Backend Circuit Size` when running `nargo info`. Once you have identified that said circuit, run the following command:

```bash
./scripts/download-srs.sh path/to/your/circuit/target/<your_circuit_name>.json
```

This will download the SRS and package it into the app binary, and you'll be ready to go!

**Why do I need to know the circuit with the highest gate count?**
The SRS is the same for all circuits, so you only need to download it once. But you only need a fraction of it according to the size of the circuit you are using. So instead of downloading the whole SRS, which is over 300MB, you can download the chunk of the SRS that is needed to prove the biggest circuit you plan to use. This way you will have a much smaller SRS file to store (likely less than 50MB).

**Note**: You can still download the SRS without specifying a circuit, just run `./scripts/download-srs.sh` without any argument. This will download the fraction of the SRS needed for a circuit of up to 512k constraints, which should be enough in most cases.

### You don't know which circuits you will use for now and just want to try things out

Then you can skip the process described above and the app will revert to fetching the SRS from Aztec's server. This is the default strategy used in the app. This approach will slow down the proof generation process, especially if you have a slow network connection. Also it is not recommended for production as you should not expect users to have a fast connection at all times and this may severely impact their data plan without them realizing it.

## How to replace the circuit

This app comes with a basic Noir circuit checking that the prover knows two private inputs `a` and `b` such that the public input `result` is equal to their product `a * b`, a circuit verifying a secp256r1 signature and one doing multiple rounds of pedersen hashing. You can replace any of these circuits with your own by following these steps:

1. Go into the `circuits` folder
2. Create a new folder for your circuit such as `my_circuit`
3. Create a `Nargo.toml` file in this folder following the structure of the `Nargo.toml` file in the other subfolders of the `circuits` folder. Don't forget to change the name of the circuit in the `name` field
4. Create a `src` folder and create a `main.nr` file in it
5. Make sure you have the version 0.30.0 of `nargo`. You can check by running `nargo --version`. If you have a different version, you can use `noirup -v 0.30.0`. And if you don't have `noirup` follow the instructions [here](https://noir-lang.org/docs/getting_started/installation/).
6. Write your Noir code in `main.nr` and run `nargo check` to generate the `Prover.toml` and `Verifier.toml` files
7. Run `nargo compile` to compile the circuit
8. It will generate a new `<your_circuit_name>.json` file in `/target`
9. You can then replace the import in the Javascript code to load this circuit instead

## Note on performance

Bear in mind that mobile phones have a limited amount of available RAM. The circuit used in this app is really simple so the memory usage is not a problem. However, if you plan to use more complex circuits, you should be aware that the memory usage will increase and may go above the available memory on the device causing the proof generation to fail.

## A note on Honk

While still a work of progress, Honk APIs are already exposed in Barretenberg and this app gives the ability to tap into it. You can switch between Honk and UltraPlonk (current proofs used by Noir) by specifying the `proofType` of the prove and verify functions. Specify `honk` to use Honk and `plonk` to use UltraPlonk. If not specified, the default is UltraPlonk.

You will notice Honk is substantially faster than UltraPlonk, and uses less memory than UltraPlonk. However, it is still in development and there is no fully working on-chain verifier for it at the moment.

## Noir version currently supported

The current version of Noir supported by the app is 0.30.0
