---
label: Basics
order: 500
---

# Basics

---

## Install - Create - Run

Install SDK

```
sh -ci "$(curl -fsSL https://sdk.dfinity.org/install.sh)"
```

Get version

```
dfx --version
```

Create a new project

```
dfx new hello
```

Start local network - the flag is for running on background

```
dfx start --background
```

Stop local network

```
dfx stop
```

Upgrade dfx version

```
dfx upgrade
```

Get Candid uri ulr

```
echo "http://localhost:8000/?canisterId=$(dfx canister id __Candid_UI)"
```
