# How to setup a iota-core network with 2 nodes (1 bootstrap/validator, 1 peer)

## Prerequirements

This tutorial assumes that all the required build tools (git, go 1.22, etc) are installed on both nodes.

## Remarks

All processes are built from the develop branch.

All the settings (keys, seeds, adresses,...) are retrieved from the file <b>iota-core/tools/docker-network/docker-compose.yml</b>

## Step 1: build all the needed processes on both nodes

### On the bootstrap node (Validator)

#### iota-core

```bash
git clone https://github.com/iotaledger/iota-core.git
cd iota-core
go build -tags=rocksdb
```

#### inx-indexer

```bash
git clone https://github.com/iotaledger/inx-indexer.git
cd inx-indexer
go build
```

#### inx-blockissuer

```bash
git clone https://github.com/iotaledger/inx-blockissuer.git
cd inx-blockissuer
go build 
```

#### inx-validator

```bash
git clone https://github.com/iotaledger/inx-validator.git
cd inx-validator
go build 
```

#### inx-faucet

```bash
git clone https://github.com/iotaledger/inx-faucet.git
cd inx-faucet
go build
```

#### inx-dashboard

```bash
git clone https://github.com/iotaledger/inx-dashboard.git
cd inx-dashboard
go build
```

### On the peer node

#### iota-core

```bash
git clone https://github.com/iotaledger/iota-core.git
cd iota-core
go build -tags=rocksdb
```

## Step 2: create genesis snapshot

The docker config is used as template.

The docker config has 4 validator nodes as default so we have to remove 3 for our setup.

This is done in <b>iota-core/tools/genesis-snapshot/presets/presets.go</b>:

```golang
...
var (
    AccountsDockerFunc = func(protoParams iotago.ProtocolParameters) []snapshotcreator.AccountDetails {
        return []snapshotcreator.AccountDetails{
            {
                /*
                    node-01-validator

                    Ed25519 Public Key:   293dc170d9a59474e6d81cfba7f7d924c09b25d7166bcfba606e53114d0a758b
                    Ed25519 Address:      rms1qzg8cqhfxqhq7pt37y8cs4v5u4kcc48lquy2k73ehsdhf5ukhya3ytgk0ny
                    Account Address:      rms1pzg8cqhfxqhq7pt37y8cs4v5u4kcc48lquy2k73ehsdhf5ukhya3y5rx2w6
                    Restricted Address:   rms1xqqfqlqzayczurc9w8cslzz4jnjkmrz5lurs32m68x7pkaxnj6unkyspqg8mulpm, Capabilities: mana
                */
                AccountID:            blake2b.Sum256(lo.PanicOnErr(hexutil.DecodeHex("0x293dc170d9a59474e6d81cfba7f7d924c09b25d7166bcfba606e53114d0a758b"))),
                Address:              iotago.Ed25519AddressFromPubKey(lo.PanicOnErr(hexutil.DecodeHex("0x293dc170d9a59474e6d81cfba7f7d924c09b25d7166bcfba606e53114d0a758b"))),
                Amount:               mock.MinValidatorAccountAmount(protoParams),
                IssuerKey:            iotago.Ed25519PublicKeyHashBlockIssuerKeyFromPublicKey(ed25519.PublicKey(lo.PanicOnErr(hexutil.DecodeHex("0x293dc170d9a59474e6d81cfba7f7d924c09b25d7166bcfba606e53114d0a758b")))),
                ExpirySlot:           iotago.MaxSlotIndex,
                BlockIssuanceCredits: iotago.MaxBlockIssuanceCredits / 5,
                StakingEndEpoch:      iotago.MaxEpochIndex,
                FixedCost:            1,
                StakedAmount:         mock.MinValidatorAccountAmount(protoParams),
                Mana:                 iotago.Mana(mock.MinValidatorAccountAmount(protoParams)),
            },
            // !!!! remove the following 3 validators
            // {
            // 	/*
            // 		node-02-validator

            // 		Ed25519 Public Key:   05c1de274451db8de8182d64c6ee0dca3ae0c9077e0b4330c976976171d79064
            // 		Ed25519 Address:      rms1qqm4xk8e9ny5w5rxjkvtp249tfhlwvcshyr3pc0665jvp7g3hc875flpz2p
            // 		Account Address:      rms1pqm4xk8e9ny5w5rxjkvtp249tfhlwvcshyr3pc0665jvp7g3hc875k538hl
            // 		Restricted Address:   rms1xqqrw56clykvj36sv62e3v9254dxlaenzzuswy8plt2jfs8ezxlql6spqgkulf7u, Capabilities: mana
            // 	*/
            // 	AccountID:            blake2b.Sum256(lo.PanicOnErr(hexutil.DecodeHex("0x05c1de274451db8de8182d64c6ee0dca3ae0c9077e0b4330c976976171d79064"))),
            // 	Address:              iotago.Ed25519AddressFromPubKey(lo.PanicOnErr(hexutil.DecodeHex("0x05c1de274451db8de8182d64c6ee0dca3ae0c9077e0b4330c976976171d79064"))),
            // 	Amount:               mock.MinValidatorAccountAmount(protoParams),
            // 	IssuerKey:            iotago.Ed25519PublicKeyHashBlockIssuerKeyFromPublicKey(ed25519.PublicKey(lo.PanicOnErr(hexutil.DecodeHex("0x05c1de274451db8de8182d64c6ee0dca3ae0c9077e0b4330c976976171d79064")))),
            // 	ExpirySlot:           iotago.MaxSlotIndex,
            // 	BlockIssuanceCredits: iotago.MaxBlockIssuanceCredits / 5,
            // 	StakingEndEpoch:      iotago.MaxEpochIndex,
            // 	FixedCost:            1,
            // 	StakedAmount:         mock.MinValidatorAccountAmount(protoParams),
            // 	Mana:                 iotago.Mana(mock.MinValidatorAccountAmount(protoParams)),
            // },
            // {
            // 	/*
            // 		node-03-validator

            // 		Ed25519 Public Key:   1e4b21eb51dcddf65c20db1065e1f1514658b23a3ddbf48d30c0efc926a9a648
            // 		Ed25519 Address:      rms1qp4wuuz0y42caz48vv876qfpmffswsvg40zz8v79sy8cp0jfxm4kuvz0a44
            // 		Account Address:      rms1pp4wuuz0y42caz48vv876qfpmffswsvg40zz8v79sy8cp0jfxm4kunflcgt
            // 		Restricted Address:   rms1xqqx4mnsfuj4tr525a3slmgpy8d9xp6p3z4ugganckqslq97fymwkmspqgnzrkjq, Capabilities: mana
            // 	*/
            // 	AccountID:            blake2b.Sum256(lo.PanicOnErr(hexutil.DecodeHex("0x1e4b21eb51dcddf65c20db1065e1f1514658b23a3ddbf48d30c0efc926a9a648"))),
            // 	Address:              iotago.Ed25519AddressFromPubKey(lo.PanicOnErr(hexutil.DecodeHex("0x1e4b21eb51dcddf65c20db1065e1f1514658b23a3ddbf48d30c0efc926a9a648"))),
            // 	Amount:               mock.MinValidatorAccountAmount(protoParams),
            // 	IssuerKey:            iotago.Ed25519PublicKeyHashBlockIssuerKeyFromPublicKey(ed25519.PublicKey(lo.PanicOnErr(hexutil.DecodeHex("0x1e4b21eb51dcddf65c20db1065e1f1514658b23a3ddbf48d30c0efc926a9a648")))),
            // 	ExpirySlot:           iotago.MaxSlotIndex,
            // 	BlockIssuanceCredits: iotago.MaxBlockIssuanceCredits / 5,
            // 	StakingEndEpoch:      iotago.MaxEpochIndex,
            // 	FixedCost:            1,
            // 	StakedAmount:         mock.MinValidatorAccountAmount(protoParams),
            // 	Mana:                 iotago.Mana(mock.MinValidatorAccountAmount(protoParams)),
            // },
            // {
            // 	/*
            // 		node-04-validator

            // 		Ed25519 Public Key:   c9ceac37d293155a578381aa313ee74edfa3ac73ee930d045564aae7771e8ffe
            // 		Private Key: 5cceed8ca18146639330177ab4f61ab1a71e2d3fea3d4389f9e2e43f34ec8b33c9ceac37d293155a578381aa313ee74edfa3ac73ee930d045564aae7771e8ffe
            // 		Account Address:      rms1pr8cxs3dzu9xh4cduff4dd4cxdthpjkpwmz2244f75m0urslrsvtsshrrjw
            // 		Ed25519 Address:      rms1qr8cxs3dzu9xh4cduff4dd4cxdthpjkpwmz2244f75m0urslrsvts0unx0s
            // 		Restricted Address:   rms1xqyvnn4vxlffx92627pcr2338mn5ahar43e7aycdq32kf2h8wu0gllspqgz9eyua, Capabilities: mana
            // 	*/
            // 	AccountID:            blake2b.Sum256(lo.PanicOnErr(hexutil.DecodeHex("0xc9ceac37d293155a578381aa313ee74edfa3ac73ee930d045564aae7771e8ffe"))),
            // 	Address:              iotago.Ed25519AddressFromPubKey(lo.PanicOnErr(hexutil.DecodeHex("0xc9ceac37d293155a578381aa313ee74edfa3ac73ee930d045564aae7771e8ffe"))),
            // 	Amount:               mock.MinValidatorAccountAmount(protoParams),
            // 	IssuerKey:            iotago.Ed25519PublicKeyHashBlockIssuerKeyFromPublicKey(ed25519.PublicKey(lo.PanicOnErr(hexutil.DecodeHex("0xc9ceac37d293155a578381aa313ee74edfa3ac73ee930d045564aae7771e8ffe")))),
            // 	ExpirySlot:           iotago.MaxSlotIndex,
            // 	BlockIssuanceCredits: iotago.MaxBlockIssuanceCredits / 5,
            // 	StakingEndEpoch:      iotago.MaxEpochIndex,
            // 	FixedCost:            1,
            // 	StakedAmount:         mock.MinValidatorAccountAmount(protoParams),
            // 	Mana:                 iotago.Mana(mock.MinValidatorAccountAmount(protoParams)),
            // },
            {
                /*
                    inx-blockissuer

                    Ed25519 Private Key:  432c624ca3260f910df35008d5c740593b222f1e196e6cdb8cd1ad080f0d4e33997be92a22b1933f36e26fba5f721756f95811d6b4ae21564197c2bfa4f28270
                    Ed25519 Public Key:   997be92a22b1933f36e26fba5f721756f95811d6b4ae21564197c2bfa4f28270
                    Ed25519 Address:      rms1qrkursay9fs2qjmfctamd6yxg9x8r3ry47786x0mvwek4qr9xd9d583cnpd
                    Account Address:      rms1prkursay9fs2qjmfctamd6yxg9x8r3ry47786x0mvwek4qr9xd9d5c6gkun
                    Restricted Address:   rms1xqqwmswr5s4xpgztd8p0hdhgseq5cuwyvjhmclgeld3mx65qv5e54kspqgda0nrn, Capabilities: mana
                */
                AccountID:            blake2b.Sum256(lo.PanicOnErr(hexutil.DecodeHex("0x997be92a22b1933f36e26fba5f721756f95811d6b4ae21564197c2bfa4f28270"))),
                Address:              iotago.Ed25519AddressFromPubKey(lo.PanicOnErr(hexutil.DecodeHex("0x997be92a22b1933f36e26fba5f721756f95811d6b4ae21564197c2bfa4f28270"))),
                Amount:               mock.MinIssuerAccountAmount(protoParams),
                IssuerKey:            iotago.Ed25519PublicKeyHashBlockIssuerKeyFromPublicKey(ed25519.PublicKey(lo.PanicOnErr(hexutil.DecodeHex("0x997be92a22b1933f36e26fba5f721756f95811d6b4ae21564197c2bfa4f28270")))),
                ExpirySlot:           iotago.MaxSlotIndex,
                BlockIssuanceCredits: iotago.MaxBlockIssuanceCredits / 5,
                Mana:                 iotago.Mana(mock.MinIssuerAccountAmount(protoParams)),
            },
        }
    }
...
```


Now build the tool and generate the snapshot

```bash
cd tools/genesis-snapshot
go build -tags=rocksdb
./genesis-snapshot . --config docker --filename snapshot.bin
```

The snapshot file has to be copied on both nodes into <b>iota-core/testnet</b>.

## Step 3: start the processes on the nodes

### On the bootstrap node (validator)

<b>iota-core/config_defaults.json</b> is used as config. In this file the inx interface has to be enabled:

```json
...
"inx": {
    "enabled": true,
    "bindAddress": "localhost:9029"
}
...
```

Then start all the processes (use separate shells for each process):

```bash
./iota-core -c config_defaults.json
```

```bash
./inx-indexer
```

```bash
export "BLOCKISSUER_PRV_KEY=432c624ca3260f910df35008d5c740593b222f1e196e6cdb8cd1ad080f0d4e33997be92a22b1933f36e26fba5f721756f95811d6b4ae21564197c2bfa4f28270"
./inx-blockissuer --inx.address=127.0.0.1:9029 --blockIssuer.accountAddress=rms1prkursay9fs2qjmfctamd6yxg9x8r3ry47786x0mvwek4qr9xd9d5c6gkun --blockIssuer.proofOfWork.targetTrailingZeros=5
```

```bash
export "VALIDATOR_PRV_KEY=443a988ea61797651217de1f4662d4d6da11fd78e67f94511453bf6576045a05293dc170d9a59474e6d81cfba7f7d924c09b25d7166bcfba606e53114d0a758b"
./inx-validator --inx.address=127.0.0.1:9029 --validator.ignoreBootstrapped=true --validator.accountAddress=rms1pzg8cqhfxqhq7pt37y8cs4v5u4kcc48lquy2k73ehsdhf5ukhya3y5rx2w6 --validator.issueCandidacyPayload=${ISSUE_CANDIDACY_PAYLOAD_V1:-true}
```

```bash
export "FAUCET_PRV_KEY=de52b9964dda96564e9fab362ab16c2669c715c6a2a853bece8a25fc58c599755b938327ea463e0c323c0fd44f6fc1843ed94daecc6909c6043d06b7152e4737"
./inx-faucet --inx.address=127.0.0.1:9029 --faucet.bindAddress=0.0.0.0:8091 --faucet.rateLimit.enabled=false --faucet.baseTokenAmount=1000000000 --faucet.baseTokenAmountSmall=100000000 --faucet.baseTokenAmountMaxTarget=5000000000 --faucet.manaAmount=100000000 --faucet.manaAmountMinFaucet=1000000000
```

```bash
./inx-dashboard --inx.address=localhost:9029 --dashboard.bindAddress=0.0.0.0:8081 --dashboard.auth.username="admin" --dashboard.auth.passwordHash="87ee6727b04474a25c5a4fe3914488d56363fa1d6cc03d03e70a424fa4d87a7c" --dashboard.auth.passwordSalt="d46309210eb66b523f3c7ac8c2f79e4856505dc0d3621acb5e79ae5f5bf27c73"
```


### On the peer node

```bash
./iota-core -c config_defaults.json ---p2p.autopeering.bootstrapPeers=/dns/192.168.178.33/tcp/15600/p2p/12D3KooWPpHpFWUFKi3s1D5BnkAdPAxfiEoLV1TQ97rj947pA3EX
```

<b>!!! ATTENTION: Use the ip and id of your bootstrap node in the above command.!!!</b>

## Step 4: check status

Now the dashboard of the validator node should be accessible on <http://127.0.0.1:8081/dashboard>
(User: admin, Password: admin)

## Further steps

To add more peers (no validator) it should be sufficient to start the iota-core process on the nodes

```bash
./iota-core -c config_defaults.json --p2p.autopeering.bootstrapPeers=/dns/192.168.178.33/tcp/15600/p2p/12D3KooWPpHpFWUFKi3s1D5BnkAdPAxfiEoLV1TQ97rj947pA3EX
```
