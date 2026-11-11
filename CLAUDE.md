# Contexto do fork (eacarva/evolution-go)

Fork do `evolution-foundation/evolution-go`, deployado no Easypanel a partir do branch `main`
(build pelo `Dockerfile` da raiz). Responder em português, de forma direta.

## Histórico (11/11/2026)

1. **405 "Client outdated" ao conectar instância** — o código buscava a versão do WhatsApp Web mas
   só gravava em `store.DeviceProps.Version` (o que aparece no celular). O handshake usa
   `store.SetWAVersion`. Corrigido em `pkg/whatsmeow/service/whatsmeow.go` (bloco após
   "Setting whatsapp version from web").
2. **whatsmeow atualizado** de `b572e5bcb92b` (jun/2026) para `c88388f983b4` (10/11/2026), que
   exige Go 1.26. Ajustadas as APIs que mudaram em `pkg/user/service/user_service.go`:
   `GetBlocklist`/`UpdateBlocklist` (novo parâmetro `dhash`, passamos `""`) e `SetStatusMessage`
   (agora recebe `types.SetStatusInput`). `types.Blocklist` trocou `JIDs` por `Items`.
3. **Conexão e texto funcionando.** Botões davam `server returned error 405`. Portado o fix de
   `lucasgiovannibr/evolution-go#74` em `pkg/sendMessage/service/send_service.go` (`SendButton`):
   reply e CTA viram `InteractiveMessage` puro com `<biz><interactive type="native_flow" v="1">
   <native_flow name="mixed" v="9"/></interactive></biz>` + `<bot biz_bot="1"/>`. Pix inalterado.
   Carrossel não manda mais título/subtítulo vazios (`carouselCardHeader`).

## Pendências / atenção

- **`go.mod`/`go.sum` NÃO foram atualizados** (o ambiente anterior não tinha acesso ao proxy de
  módulos Go). Hoje o `Dockerfile` roda `go get go.mau.fi/whatsmeow@${WHATSMEOW_REF} && go mod tidy`
  antes do build. Primeira tarefa sugerida: rodar localmente
  `go get go.mau.fi/whatsmeow@c88388f983b4 && go mod tidy && go build ./...`, commitar `go.mod`/`go.sum`
  e remover esse passo do Dockerfile.
- **Nada foi compilado nem testado localmente** depois das mudanças — rodar `go build ./...` e
  `go test ./...`.
- Validar botões no aparelho (reply 1/3, CTA copy/url/call, agrupados, Pix, carrosséis).
  O fix de referência foi testado com número WhatsApp Business.
- `/send/list`: WhatsApp recusa listas vindas de dispositivo vinculado (405/479). O fork de
  referência converte lista em botões reply como fallback — avaliar portar.
- O servidor do Easypanel ficou sem disco durante um build; rodar `docker builder prune -af`
  se acontecer de novo.
- Rotacionar `GLOBAL_API_KEY` e senha do Postgres (vazaram num log de build).
