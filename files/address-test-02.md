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
    os.Exit(1)
  }
  address.DefaultNetwork = address.Mainnet
  w, err := wallet.New("/tmp/realsim-wallet.db", seed)
  if err != nil {
    fmt.Fprintln(os.Stderr, err)
    os.Exit(1)
  }
  defer w.Close()
  addr, err := w.NextReceiveAddress()
  if err != nil {
    fmt.Fprintln(os.Stderr, err)
    os.Exit(1)
  }
  fmt.Println(addr)
}
EOF
SEED=$(openssl rand -hex 32)
echo "SEED: $SEED"
go mod tidy
go run main.go $SEED
SEED: 65b96a6bc4bfffebd01b3b75603afa2f4c9d69a5197a554d4a76a98f2f8bcfe2
bq1z9w4j4kk5pxn75ceywuvr5rtpzrg53mna6jq5jzltmd5wxgj6k8gss0zc27
ion@ion-VirtualBox:~/p2qpk-realsim-test$ 

