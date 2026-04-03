# Tutorial: Como Publicar um Pacote Go no pkg.go.dev

Este tutorial explica como criar, estruturar e publicar um pacote Go no [pkg.go.dev](https://pkg.go.dev/).

## 1. Pré-requisitos

- Go instalado (1.21+)
- Conta no GitHub (ou outro host público de Git)
- Git configurado localmente

## 2. Criar o Repositório

Crie um repositório público no GitHub. O nome do repositório será parte do import path.

Exemplo: `github.com/seu-usuario/meu-pacote`

## 3. Estrutura do Projeto

### Módulo com um único pacote

Se o repositório contém apenas um pacote, os arquivos `.go` ficam na raiz:

```bash
meu-pacote/
├── go.mod
├── go.sum
├── LICENSE
├── README.md
├── CHANGELOG.md
├── meu_pacote.go        # package meupacote
├── meu_pacote_test.go
└── example_test.go      # funções Example* (renderizadas no pkg.go.dev)
```

Import: `github.com/seu-usuario/meu-pacote`

### Módulo com múltiplos pacotes (subpacotes)

Se o repositório contém mais de um pacote, cada um fica em sua própria pasta:

```bash
meu-modulo/
├── go.mod               # module github.com/seu-usuario/meu-modulo
├── go.sum
├── LICENSE
├── README.md
├── CHANGELOG.md
├── json/
│   ├── doc.go           # package json — documentação do pacote
│   ├── unmarshal.go
│   ├── decoder.go
│   ├── json_test.go
│   └── example_test.go
└── xml/
    ├── doc.go           # package xml
    ├── unmarshal.go
    ├── decoder.go
    ├── xml_test.go
    └── example_test.go
```

Imports:

- `github.com/seu-usuario/meu-modulo/json`
- `github.com/seu-usuario/meu-modulo/xml`

### Regras importantes sobre estrutura

- O `go.mod` fica na raiz e define o module path
- Cada pasta com arquivos `.go` é um pacote separado
- O nome do pacote (declarado no `package`) não precisa ser igual ao nome da pasta, mas por convenção deve ser
- Arquivos `_test.go` ficam na mesma pasta do código que testam
- Pasta `internal/` contém código que não pode ser importado por outros módulos
- Pasta `cmd/` é usada para executáveis (CLIs), não para bibliotecas

## 4. Inicializar o Módulo

```bash
mkdir meu-pacote
cd meu-pacote
go mod init github.com/seu-usuario/meu-pacote
```

O `go.mod` gerado:

```bash
module github.com/seu-usuario/meu-pacote

go 1.24
```

A diretiva `go` define a versão mínima do Go que consumidores precisam ter.

## 5. Escrever o Código

### Arquivo principal

```go
// Package meupacote faz algo útil.
//
// Descrição mais detalhada aqui. Este comentário aparece no pkg.go.dev.
package meupacote

// MinhaFuncao faz algo.
func MinhaFuncao() string {
    return "resultado"
}
```

### doc.go (para subpacotes)

Quando o pacote está em uma subpasta, crie um `doc.go` com a documentação:

```go
// Package json provides case-sensitive JSON unmarshaling.
//
// Descrição detalhada, exemplos de uso, etc.
// Tudo isso aparece no pkg.go.dev.
package json
```

### Regras de documentação para o pkg.go.dev

- O comentário acima de `package` é a documentação do pacote
- Comentários acima de funções/tipos exportados são a documentação deles
- Use exemplos em `example_test.go` com `// Output:` para que sejam renderizados
- O `README.md` da raiz aparece na página do módulo no pkg.go.dev

## 6. Escrever Testes

```go
package meupacote_test

import (
    "testing"
    "github.com/seu-usuario/meu-pacote"
)

func TestMinhaFuncao(t *testing.T) {
    resultado := meupacote.MinhaFuncao()
    if resultado != "resultado" {
        t.Errorf("esperado 'resultado', obteve %q", resultado)
    }
}
```

### Funções Example (aparecem no pkg.go.dev)

```go
package meupacote_test

import (
    "fmt"
    "github.com/seu-usuario/meu-pacote"
)

func ExampleMinhaFuncao() {
    resultado := meupacote.MinhaFuncao()
    fmt.Println(resultado)
    // Output:
    // resultado
}
```

O comentário `// Output:` é obrigatório para que o exemplo seja executado como teste
e renderizado no pkg.go.dev.

## 7. Licença

Inclua um arquivo `LICENSE` na raiz. O pkg.go.dev exibe a licença na página do módulo.
Licenças comuns: MIT, Apache 2.0, BSD-3-Clause.

## 8. Versionamento

O Go usa [Semantic Versioning](https://semver.org/) via git tags.

### Formato da versão

```bash
vMAJOR.MINOR.PATCH
```

- MAJOR: mudanças incompatíveis na API (breaking changes)
- MINOR: novas funcionalidades compatíveis com versões anteriores
- PATCH: correções de bugs compatíveis com versões anteriores

### Criar a primeira versão

```bash
git add .
git commit -m "feat: initial release"
git tag v0.1.0
git push origin main --tags
```

### Versões seguintes

```bash
# Correção de bug
git tag v0.1.1

# Nova funcionalidade
git tag v0.2.0

# Breaking change
git tag v1.0.0
```

### Regras de versionamento no Go

- Versões `v0.x.x` são consideradas instáveis (a API pode mudar)
- A partir de `v1.0.0`, a API é considerada estável
- Para `v2.0.0+`, o module path deve incluir o major version:
  - `go.mod`: `module github.com/seu-usuario/meu-pacote/v2`
  - Import: `github.com/seu-usuario/meu-pacote/v2`
- Nunca delete ou mova tags já publicadas

### Pre-releases

```bash
git tag v0.1.0-alpha.1
git tag v0.1.0-beta.1
git tag v0.1.0-rc.1
```

Pre-releases não são instaladas por padrão com `go get`.

## 9. Publicar no pkg.go.dev

O pkg.go.dev indexa automaticamente a partir do Go Module Proxy. Para acionar a indexação:

### Opção 1: Acessar a URL diretamente

Abra no navegador:

```bash
https://pkg.go.dev/github.com/seu-usuario/meu-pacote
```

O pkg.go.dev vai buscar e indexar o módulo na primeira visita.

### Opção 2: Forçar via Go Proxy

```bash
GOPROXY=https://proxy.golang.org go get github.com/seu-usuario/meu-pacote@v0.1.0
```

### Opção 3: Solicitar via API

```bash
curl "https://proxy.golang.org/github.com/seu-usuario/meu-pacote/@v/v0.1.0.info"
```

Após qualquer uma dessas opções, o pacote aparece no pkg.go.dev em alguns minutos.

## 10. Consumir o Pacote

Quem quiser usar o pacote:

```bash
go get github.com/seu-usuario/meu-pacote@v0.1.0
```

```go
import "github.com/seu-usuario/meu-pacote"
```

Para subpacotes:

```bash
go get github.com/seu-usuario/meu-modulo/json@v0.1.0
```

```go
import "github.com/seu-usuario/meu-modulo/json"
```

## 11. Checklist para Publicação

- [ ] `go.mod` com module path correto (`github.com/usuario/repo`)
- [ ] `LICENSE` na raiz
- [ ] `README.md` na raiz (aparece no pkg.go.dev)
- [ ] Comentários de documentação em funções e tipos exportados
- [ ] `doc.go` em subpacotes
- [ ] Funções `Example*` em `example_test.go`
- [ ] Testes passando (`go test ./...`)
- [ ] Sem dependências desnecessárias no `go.sum`
- [ ] Git tag no formato `vX.Y.Z`
- [ ] Tag publicada no remote (`git push --tags`)
- [ ] Repositório público

## 12. Boas Práticas

- Mantenha um `CHANGELOG.md` seguindo o formato [Keep a Changelog](https://keepachangelog.com/)
- Use CI (GitHub Actions) para rodar testes e lint automaticamente
- Comece com `v0.x.x` até a API estabilizar
- Documente breaking changes claramente no CHANGELOG
- Use `go vet` e `golangci-lint` antes de cada release
- Rode `go mod tidy` para limpar dependências não utilizadas
