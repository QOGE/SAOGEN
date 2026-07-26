ion@ion-VirtualBox:~$ mkdir -p ~/p2qpk-realsim-test
cd ~/p2qpk-realsim-test
cat > go.mod << 'EOF'
module p2qpk-realsim-test

go 1.21

require github.com/saogen/qoge-sphincs-wallet v0.0.0

replace (
  github.com/saogen/qoge-sphincs-wallet => /home/ion/symbiont-wallet
  github.com/open-quantum-safe/liboqs-go => /home/ion/liboqs-go
)
EOF
cat > main.go << 'EOF'
package main

import (
  "encoding/hex"
  "fmt"
  "os"

  "github.com/saogen/qoge-sphincs-wallet/address"
  "github.com/saogen/qoge-sphincs-wallet/wallet"
)

func main() {
  seed, err := hex.DecodeString(os.Args[1])
  if err != nil || len(seed) != 32 {
    fmt.Fprintf(os.Stderr, "seed must be 32 bytes hex, got %d bytes\n", len(seed))
go run main.go $SEED-hex 32)err)ess()wallet.db", seed)
SEED: 062b10927bddf8939ad51240c4ec95d5c5c55d2550b3a9affcd883dc2313d50a
bq1z9w4j4kk5pxn75ceywuvr5rtpzrg53mna6jq5jzltmd5wxgj6k8gss0zc27
ion@ion-VirtualBox:~/p2qpk-realsim-test$ 

