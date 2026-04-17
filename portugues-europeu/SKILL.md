---
name: portugues-europeu
description: Garante que toda a comunicação com o utilizador no Paperclip é feita em português europeu (pt-PT). Aplica-se a respostas, mensagens, explicações e qualquer output dirigido ao utilizador. Termos técnicos em inglês só quando a tradução prejudicar a compreensão ou quando o termo já é convenção na indústria.
---

# Português europeu na comunicação com o utilizador

Toda a comunicação com o utilizador no Paperclip tem de ser em **português europeu (pt-PT)**. Isto inclui respostas, explicações, confirmações, mensagens de erro e qualquer texto apresentado ao utilizador.

## Regras de uso

### Usar sempre pt-PT, nunca pt-BR

Vocabulário e ortografia têm de seguir o padrão europeu. Exemplos:

| Usar (pt-PT) | Evitar (pt-BR) |
|---|---|
| ficheiro | arquivo |
| ecrã | tela |
| utilizador | usuário |
| rato | mouse (quando refere periférico) |
| a equipa | o time |
| estás a fazer | você está fazendo |
| telemóvel | celular |
| autocarro | ônibus |

Gerúndios devem ser substituídos por "a + infinitivo" sempre que natural:
- ✅ "estou a processar o pedido"
- ❌ "estou processando o pedido"

Uso da segunda pessoa: preferir `tu` ou construções impessoais. Evitar `você` em registo informal.

### Termos técnicos: quando manter em inglês

Mantém em inglês apenas quando:

1. **A tradução não é standard na indústria** e cria ambiguidade
   - ✅ endpoint, webhook, cache, token, payload, middleware, rate limit
   - ✅ pull request, merge, commit, branch, rebase
   - ✅ log, deploy, build, rollback, feature flag

2. **É nome próprio ou identificador técnico** (nomes de APIs, serviços, bibliotecas, variáveis)
   - ✅ "o `UserService` devolve um 401"
   - ✅ "chama o endpoint `/api/v2/articles`"

3. **A tradução existe mas é pouco usada em contexto profissional**
   - ✅ "bug", "stack trace", "race condition", "edge case"

### Termos técnicos: quando traduzir

Traduz quando a versão portuguesa é natural e amplamente usada:

- ficheiro (file)
- pasta (folder)
- pedido / resposta (request / response)
- base de dados (database)
- servidor (server)
- rede (network)
- ligação (connection)
- erro (error)
- chave / valor (key / value)

### Mistura controlada

É aceitável — e muitas vezes preferível — combinar: *"o endpoint devolveu um 500, o payload estava malformado"*. O objectivo é clareza, não purismo.

## Como aplicar

1. Antes de enviar qualquer resposta ao utilizador, verifica se o texto está em pt-PT.
2. Se o utilizador escrever em pt-BR ou em inglês, **responde na mesma em pt-PT** — não adoptes o registo do input. Só muda de língua se o utilizador a mudar deliberadamente (ex: escreve toda a mensagem em inglês ou pede explicitamente outra língua).
3. Ao citar conteúdo externo (documentação, logs, erros), podes manter o original, mas o texto envolvente tem de estar em pt-PT.
4. Em dúvida entre traduzir ou não um termo técnico: pergunta-te se um programador português sénior usaria a palavra traduzida numa conversa de trabalho. Se sim, traduz. Se não, mantém em inglês.

## Não se aplica a

- Código-fonte, nomes de variáveis, identificadores de API — ficam sempre em inglês.
- Citações literais de erros, logs ou documentação externa.
- Quando o utilizador pede explicitamente resposta noutra língua.
