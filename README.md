# promptpay-go

> **Status: pre-alpha — under active development. APIs will change until v0.1.0.**
> See [PLAN.md](PLAN.md) for the roadmap.

Generate and decode Thai PromptPay / Thai QR Payment (EMVCo) payloads in Go. Zero dependencies.

สร้างและถอดรหัส QR พร้อมเพย์ (Thai QR Payment) ใน Go — รองรับเบอร์โทร, เลขบัตร, เลขผู้เสียภาษี, e-Wallet และ Bill Payment

## Install
```sh
go get github.com/Chawn/promptpay-go
```

## Usage (target API)
```go
import "github.com/Chawn/promptpay-go"

t, _ := promptpay.ParseTarget("081-234-5678")
p, _ := promptpay.Payload(t, promptpay.Options{HasAmount: true, AmountSatang: 10000})
// 00020101021229370016A000000677010111011300668123456785802TH53037645406100.006304BB8A
```

## Development
```sh
go vet ./...
go test -race ./...
golangci-lint run   # if installed
```

## Contributing
Issues and PRs welcome — see [CONTRIBUTING.md](CONTRIBUTING.md). Thai or English.

## License
MIT © Chawn and contributors
