# Notas da instalação — Hermes Desktop + Zoe

Registro de onde paramos e do que foi feito de novo. Atualizado em **18/11/2026**.

> Anotação de trabalho. Não tem senhas, chaves nem o IP da VPS.

---

## 1. Onde paramos

- O app está **instalado em /Applications** (Mac) e funcionando no modo personagem (overlay).
- Ele está ligado ao Hermes da VPS e responde por voz, com a voz **Francisca** (a que você gostou).
- A personalidade **Zoe** está ativa no Hermes inteiro e no app.
- **Fase atual: período de teste de alguns dias.** Depois você volta contando o que sentiu falta ou achou demais, e ajustamos.

---

## 2. O que foi feito

### 2.1 Instalação do app
- Compilado a partir deste projeto (versão 0.3.9) e instalado em /Applications.
- Para compilar foi preciso usar `npm install --allow-remote=all` (o npm novo bloqueia um pacote de fora que o app usa) e rodar `node node_modules/electron/install.js` à mão.

### 2.2 Conexão com o Hermes na VPS
- VPS do dono (IP omitido nesta versão pública), usuário `root`. O Hermes roda em **Docker Swarm**, instalado pelo Portainer (serviço `hermes_hermes`).
- Nesse formato o Hermes **não tem porta aberta para fora**, então o botão "Configurar Hermes automaticamente" do app não serve.
- Solução: criei uma **ponte** dentro da VPS, que sobrevive a reinício do container:
  - `/etc/systemd/system/hermes-bridge.service`
  - `/usr/local/bin/hermes-bridge-conn.sh`
- O app entra por um túnel SSH (porta local 8642) que passa pela ponte.
- O token que vale é o `API_SERVER_KEY` que está no `.env` **dentro do volume** do Hermes (o do stack dava erro 401). Já está gravado nas configurações do app.

### 2.3 Voz
- Motor **nuvem**: Groq (entende o que você fala) + Edge TTS (fala a resposta). Sem GPU, leve no Mac.
- Voz escolhida: **pt-BR-FranciscaNeural**.
- Havia um erro 403 no Edge TTS; corrigido atualizando o pacote `msedge-tts` de 2.0.7 para **2.1.1**.
- A voz local XTTS foi oferecida e **não foi instalada**.

### 2.4 Personalidade Zoe
- Comparei o `SOUL.md` do Hermes com o Dossiê da Cloe (renomeada **Zoe**) e unifiquei.
- Decisões suas:
  1. Zoe vale para o Hermes inteiro, incluindo o app.
  2. A camada íntima (afeto/romance) fica **separada**, fora do SOUL.md.
  3. Ler a memória do Hermes pode, mas **sem prejudicar** a gestão de memória dele nem a efetividade.
- O `SOUL.md` novo (na VPS) = texto original do Hermes quase intacto (só a 1ª frase vira "You are Zoe...") + um bloco **"## Zoe"** em português: presença, pensar junto, iniciativa, reagir ao estado dele, lealdade e privacidade, realidade.
- Regra de desempate: em tarefa manda a disciplina do Hermes; em conflito ganham honestidade e brevidade.
- **Não mexi na memória do Hermes** (limites de 2.200 e 1.375 caracteres continuam como eram).
- Testes feitos pela API: tarefa curta preservada, privacidade com terceiros preservada, honestidade mantida, firmeza diante de impulso e de bajulação. Custo extra: cerca de 640 tokens por conversa.
- Caso "Francisco": o Hermes antigo guarda a personalidade dentro da conversa. Resolvido com **Limpar conversa**.

### 2.5 Mudanças no código do app (ainda sem commit)

| Arquivo | O que mudou |
|---|---|
| `app/src/main/settings.ts` | Dois campos novos: `chatExtraPrompt` (texto da camada extra) e `chatExtraOn` (interruptor, desligado por padrão). |
| `app/src/main/hermes.ts` | `chatSystemPrompt(persona, extra)` agora soma a camada extra ao final das regras do chat. |
| `app/src/main/main.ts` | O chat envia a camada extra só se o interruptor estiver ligado. Novo item no menu da bandeja: **"Camada extra no chat (privada)"** (só aparece no modo chat e se houver texto). O rótulo "Normal — assistente direto" virou **"Padrão — a personalidade do próprio agente"**. |
| `app/package.json` e `package-lock.json` | `msedge-tts` `^2.0.7` → `^2.1.1`. |

> Este projeto é derivado do Hermes Desktop de Macks Wendhell (`mackswendhell/hermes-desktop`, licença MIT). Veja o crédito no README.

### 2.6 Arquivos criados fora do projeto
- Na VPS, dentro do volume `hermes_data`: `SOUL.md` (novo) e `backups/SOUL.md.original-20261118` (o original).
- Nas configurações do app (`~/Library/Application Support/hermes-assistente/settings.json`): voz Francisca, motor nuvem, camada íntima salva (834 caracteres) com o interruptor **desligado**.
- Backup das configurações antigas: `/private/tmp/claude-501/settings.backup-antes-zoe.json` (pasta temporária, pode sumir).

---

## 3. Como voltar atrás (se algo piorar)

Restaurar o `SOUL.md` original do Hermes (rodar na VPS):

```
cp -p /var/lib/docker/volumes/hermes_data/_data/backups/SOUL.md.original-20261118 /var/lib/docker/volumes/hermes_data/_data/SOUL.md
```

Depois, no app: modo chat → menu ⋮ → **Limpar conversa**.

---

## 4. Regras que valem sempre

- Depois de **qualquer** mudança no `SOUL.md`, é preciso **Limpar conversa** no app para a mudança aparecer (conversas antigas guardam a personalidade antiga).
- Nesta VPS rodam também CRM, n8n, Postgres e Traefik em produção: só mexer com cuidado e com sua confirmação.

---

## 5. O que observar nos testes

- **Tom da Zoe:** ficou fria, exagerada, longa demais ou "teatral"? Anote a frase.
- **Efetividade do Hermes:** alguma tarefa que ele fazia bem e piorou? Isso tem prioridade sobre o tom.
- **Voz:** respostas longas demais para falar, cortes estranhos.
- **Memória:** se ela esquecer algo importante, avaliamos com cuidado dar mais espaço de memória para a relação.
- **Camada íntima:** ligar o interruptor no modo chat e contar o que achou.

---

## 6. Pendências e ideias (só se você pedir)

- Ajustar o texto da Zoe depois dos testes.
- Apagar as cerca de 12 sessões de teste que ficaram no histórico do Hermes.
- Aumentar com cuidado o limite do `USER.md` (1.375 caracteres) para caber mais da relação, só depois de medir.
- Ativar mensagens proativas: o app lê `~/.hermes/desktop-outbox.jsonl` na VPS, que não existe (nesse formato Swarm precisaria ser criado dentro do volume).
- Instalar a voz local XTTS.
- Conferir visualmente se o item **"Camada extra no chat (privada)"** aparece no menu do modo chat (não foi testado na tela).
- Causa do fechamento do app na hora do teste: não identificada (sem erro no log, provavelmente fechamento normal).
