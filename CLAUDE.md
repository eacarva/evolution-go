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
4. **Botões davam 200 mas não apareciam no iPhone** (remetente Business, destinatário comum).
   Mesmo problema das issues #59/#71 do repo oficial. Testado no iPhone em 11/11/2026, todos aparecem:
   - Reply, URL, call e Pix: native flow com o `<biz>` de `tulir/whatsmeow#1235` (não mergeado):
     attrs `actual_actors`/`host_storage`/`privacy_mode_ts` + `<quality_control>` com `decision_id`,
     **sem `<bot>`** (`interactiveBizNodes`; "mixed" v9, Pix "payment_info" v1).
   - Copy (sozinho ou agrupado): o iPhone descarta a mensagem inteira com `cta_copy` em qualquer
     variação de `<biz>` testada, mas mostra dentro de carrossel. Vai como carrossel de 1 card
     (`buttonAsCarousel`).
   - Lista: `ListMessage` dá 405 e `single_select` não aparece. Vira botões reply, 3 por mensagem.
   - Não funcionou: formato da `oxidezap/whatsapp-rust` (com `<bot>`, `privacy_mode_ts` deslocado,
     `quality_control` vazio) — nada chegou; mandar sem `<biz>` — nada chegou.

## Teste local (sem deploy)

- Apagado em 11/11/2026 para não ocupar o Mac. Para recriar: Postgres em container
  (`docker run -d --name evogo-pg -e POSTGRES_PASSWORD=root -p 5433:5432 postgres:16-alpine` e criar
  `evogo_auth`/`evogo_users`), `.env` a partir do `.env.example` apontando para a porta 5433,
  `make dev`, ativar licença em `/license/register` (+ `/license/activate?code=...`) e ler o QR no
  manager (`http://localhost:8080/manager`). Ao terminar, `DELETE /instance/logout` antes de apagar.
- Um envio chegou no aparelho quando o log tem `Receipt ... from 248022114033783...` (o destinatário);
  recibo não garante que apareceu na tela.

## Pendências / atenção

- ~~`go.mod`/`go.sum` desatualizados~~ — resolvido: whatsmeow `c88388f983b4` pinado no `go.mod`,
  passo `go get` removido do `Dockerfile`. `go build ./...` e `go test ./...` passam localmente.
- Não testado: Android, WhatsApp Web, grupos, clique nos botões voltando no webhook.
- O servidor do Easypanel ficou sem disco durante um build; rodar `docker builder prune -af`
  se acontecer de novo.
- Rotacionar `GLOBAL_API_KEY` e senha do Postgres (vazaram num log de build).
