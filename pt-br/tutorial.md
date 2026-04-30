# Como Publicar um Pacote Go no pkg.go.dev

Se você escreveu algo em Go que pode ser útil para outras pessoas (ou para você mesmo em outros projetos), vale a pena publicar como pacote no [pkg.go.dev](https://pkg.go.dev/). O processo é mais simples do que parece. Este tutorial cobre desde a estrutura do projeto até o momento em que o pacote aparece indexado.

## 1. Pré-requisitos

- Go instalado (1.21+)
- Conta no GitHub (ou outro host público de Git)
- Git configurado localmente

## 2. Criar o Repositório

O primeiro passo é criar um repositório público no GitHub. O nome do repositório vai compor o import path do seu módulo, então escolha algo descritivo.

Exemplo: `github.com/seu-usuario/meu-pacote`

## 3. Estrutura do Projeto

A forma como você organiza os arquivos depende da complexidade do que está construindo.

### Módulo com um único pacote

Quando o repositório tem um único pacote, tudo fica na raiz. Simples assim:

```
meu-pacote/
├── go.mod
├── go.sum
├── LICENSE
├── README.md
├── CHANGELOG.md
├── meu_pacote.go          # package meupacote
├── meu_pacote_test.go
├── example_test.go         # funções Example* (renderizadas no pkg.go.dev)
└── internal/
    └── helpers.go          # código interno, não exportado para outros módulos
```

Import: `github.com/seu-usuario/meu-pacote`

### Módulo com múltiplos pacotes

Quando o módulo oferece mais de um pacote, cada um vive em sua própria pasta. A raiz continua com o `go.mod` e os arquivos de documentação:

```
meu-modulo/
├── go.mod                  # module github.com/seu-usuario/meu-modulo
├── go.sum
├── LICENSE
├── README.md
├── CHANGELOG.md
├── internal/
│   └── shared.go           # código compartilhado internamente
├── json/
│   ├── doc.go              # package json — documentação do pacote
│   ├── unmarshal.go
│   ├── decoder.go
│   ├── json_test.go
│   └── example_test.go
└── xml/
    ├── doc.go              # package xml
    ├── unmarshal.go
    ├── decoder.go
    ├── xml_test.go
    └── example_test.go
```

Imports:

- `github.com/seu-usuario/meu-modulo/json`
- `github.com/seu-usuario/meu-modulo/xml`

### Convenções de estrutura

Algumas regras que vale internalizar:

- O `go.mod` fica na raiz e define o module path.
- Cada pasta com arquivos `.go` é um pacote separado.
- O nome do pacote (declarado no `package`) não precisa ser igual ao nome da pasta, mas por convenção deve ser.
- Arquivos `_test.go` ficam na mesma pasta do código que testam.
- A pasta `internal/` é especial: o Go impede que código dentro dela seja importado por módulos externos. Use para lógica auxiliar que não faz parte da API pública.
- A pasta `cmd/` é usada para executáveis (CLIs), não para bibliotecas.

Sobre os arquivos de documentação auxiliar (`LICENSE`, `README.md`, `CHANGELOG.md`), eles são abordados nas seções de [Licença](#7-licença) e [Boas Práticas](#11-boas-práticas).

## 4. Inicializar o Módulo

Com o repositório criado, inicialize o módulo:

```bash
mkdir meu-pacote && cd meu-pacote
go mod init github.com/seu-usuario/meu-pacote
```

Isso gera o `go.mod`:

```
module github.com/seu-usuario/meu-pacote

go 1.24
```

A diretiva `go` indica a versão mínima que quem consumir o pacote precisa ter instalada.

## 5. Escrever o Código

Aqui é onde a coisa fica interessante. Vamos ver como estruturar o código pensando tanto na funcionalidade quanto na documentação que vai aparecer no pkg.go.dev.

### Documentação do pacote (doc.go)

No Go, o comentário logo acima da declaração `package` é a documentação do pacote. O pkg.go.dev renderiza esse texto na página principal. Você pode colocar esse comentário em qualquer arquivo `.go`, mas a convenção é criar um arquivo chamado `doc.go` dedicado a isso. Dessa forma a documentação fica separada da lógica e fácil de encontrar:

```go
// doc.go

// Package meupacote faz algo útil.
//
// Descrição mais detalhada aqui. Este comentário aparece
// diretamente na página do pacote no pkg.go.dev.
package meupacote
```

Isso vale tanto para o pacote raiz quanto para subpacotes. Em um módulo com múltiplos pacotes, cada subpasta teria seu próprio `doc.go`:

```go
// doc.go dentro de json/

// Package json provides case-sensitive JSON unmarshaling.
//
// Descrição detalhada, exemplos de uso, etc.
// Tudo isso aparece no pkg.go.dev.
package json
```

### Código

Com a documentação no `doc.go`, os outros arquivos ficam focados na implementação:

```go
// meu_pacote.go

package meupacote

// MinhaFuncao retorna uma string de exemplo.
func MinhaFuncao() string {
    return "resultado"
}
```

### Como a documentação funciona no pkg.go.dev

O pkg.go.dev extrai a documentação diretamente do código-fonte. Não existe um arquivo de configuração separado. As regras são:

- O comentário acima de `package` vira a documentação do pacote.
- Comentários acima de funções, tipos e constantes exportados viram a documentação deles.
- Exemplos em `example_test.go` com o comentário `// Output:` são renderizados como exemplos interativos.
- O `README.md` da raiz aparece na página do módulo.

## 6. Escrever Testes

Testes em Go ficam no mesmo diretório do código, em arquivos com sufixo `_test.go`. Para pacotes publicados, vale usar o padrão de teste externo (com `_test` no nome do pacote), que simula como um consumidor real usaria seu código:

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

### Funções Example

Esse é um recurso que muita gente ignora, mas faz diferença. Funções que começam com `Example` são executadas como testes e aparecem como exemplos de uso no pkg.go.dev:

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

O comentário `// Output:` é obrigatório. Sem ele, o exemplo não é executado como teste e não aparece na documentação.

## 7. Licença

O pkg.go.dev exibe a licença na página do módulo, então inclua um arquivo `LICENSE` na raiz do repositório. As mais comuns no ecossistema Go são MIT, Apache 2.0 e BSD-3-Clause.

Para gerar o arquivo, o jeito mais prático é usar a interface do GitHub na criação do repositório, ou consultar [choosealicense.com](https://choosealicense.com/).

## 8. Versionamento

O Go usa [Semantic Versioning](https://semver.org/) via git tags. Cada release do seu pacote é uma tag no formato:

```
vMAJOR.MINOR.PATCH
```

- MAJOR: mudanças incompatíveis na API (breaking changes)
- MINOR: novas funcionalidades compatíveis com versões anteriores
- PATCH: correções de bugs

### Criar a primeira versão

```bash
git add .
git commit -m "feat: initial release"
git tag v0.1.0
git push origin main --tags
```

### Versões seguintes

```bash
git tag v0.1.1   # correção de bug
git tag v0.2.0   # nova funcionalidade
git tag v1.0.0   # breaking change (API estável a partir daqui)
```

### Regras de versionamento no Go

Algumas particularidades que o Go impõe:

- Versões `v0.x.x` são consideradas instáveis. A API pode mudar sem cerimônia.
- A partir de `v1.0.0`, a API é considerada estável. Quebre a compatibilidade e os consumidores vão notar.
- Para `v2.0.0` em diante, o module path precisa incluir o major version:
  - `go.mod`: `module github.com/seu-usuario/meu-pacote/v2`
  - Import: `github.com/seu-usuario/meu-pacote/v2`
- Nunca delete ou mova tags já publicadas. O Go Module Proxy cacheia as versões, e inconsistências causam problemas reais.

### Pre-releases

```bash
git tag v0.1.0-alpha.1
git tag v0.1.0-beta.1
git tag v0.1.0-rc.1
```

Pre-releases não são instaladas por padrão com `go get`, o que é útil para testar antes de oficializar.

## 9. Publicar no pkg.go.dev

Não existe um botão de "publicar". O pkg.go.dev indexa automaticamente a partir do Go Module Proxy. Você só precisa acionar essa indexação de uma das seguintes formas:

### Acessar a URL diretamente

Abra no navegador:

```
https://pkg.go.dev/github.com/seu-usuario/meu-pacote
```

Na primeira visita, o pkg.go.dev busca e indexa o módulo.

### Forçar via Go Proxy

```bash
GOPROXY=https://proxy.golang.org go get github.com/seu-usuario/meu-pacote@v0.1.0
```

### Solicitar via API

```bash
curl "https://proxy.golang.org/github.com/seu-usuario/meu-pacote/@v/v0.1.0.info"
```

Qualquer uma dessas opções funciona. O pacote aparece no pkg.go.dev em alguns minutos.

## 10. Consumir o Pacote

Para quem quiser usar o seu pacote:

```bash
go get github.com/seu-usuario/meu-pacote@v0.1.0
```

```go
import "github.com/seu-usuario/meu-pacote"
```

No caso de subpacotes:

```bash
go get github.com/seu-usuario/meu-modulo/json@v0.1.0
```

```go
import "github.com/seu-usuario/meu-modulo/json"
```

## 11. Boas Práticas

- Mantenha um `CHANGELOG.md` seguindo o formato [Keep a Changelog](https://keepachangelog.com/). Isso ajuda consumidores a entenderem o que mudou entre versões sem precisar ler commits.
- O `README.md` aparece na página do módulo no pkg.go.dev. Use para explicar o que o pacote faz, como instalar e um exemplo rápido de uso. Para referência, veja o [guia de READMEs do Make a README](https://www.makeareadme.com/).
- Use CI (GitHub Actions) para rodar testes e lint automaticamente.
- Comece com `v0.x.x` até a API estabilizar.
- Use `go vet` e `golangci-lint` antes de cada release.
- Rode `go mod tidy` para limpar dependências não utilizadas.

## 12. Checklist para Publicação

- [ ] `go.mod` com module path correto (`github.com/usuario/repo`)
- [ ] `LICENSE` na raiz
- [ ] `README.md` na raiz
- [ ] Comentários de documentação em funções e tipos exportados
- [ ] `doc.go` em subpacotes
- [ ] Funções `Example*` em `example_test.go`
- [ ] Testes passando (`go test ./...`)
- [ ] Sem dependências desnecessárias (`go mod tidy`)
- [ ] Git tag no formato `vX.Y.Z`
- [ ] Tag publicada no remote (`git push --tags`)
- [ ] Repositório público
