# Servidor de WhatsApp — Kairós (v5.9.0)

---

## ✅ v5.9.0 — pré-cobrança com intervalo humano (5 a 20 segundos)

A pré-cobrança **não manda mais tudo de uma vez**. Depois de enviar para um
cliente, o servidor espera um tempo **aleatório entre 5 e 20 segundos** e só
depois manda para o próximo:

```
Cliente 1 → envia
  espera 5s
Cliente 2 → envia
  espera 13s
Cliente 3 → envia
  espera 8s
Cliente 4 → envia
  ...
```

O tempo nunca passa de 20 segundos, então a fila anda rápido, e nunca é o mesmo
ritmo — é justamente o ritmo fixo que o WhatsApp identifica como robô. Vale para
o envio automático das 07:00 e das 16:00 e também para o botão **Enviar agora**.

Nada muda no texto das mensagens, nem no controle de quem já foi avisado: se o
serviço cair no meio da fila, ele retoma de onde parou sem repetir ninguém.

**O que fazer:** publique esta pasta de novo e confira na aba **Conexão** se
aparece **Versão 5.9.0**. Sem QR Code — a sessão continua a mesma. Enquanto o
servidor estiver em versão antiga, os disparos continuam com 2 segundos fixos.

---

## ✅ v5.8.0 — pré-cobrança automática (5 dias, 2 dias e no dia)

Avisos de vencimento enviados sozinhos, **só para parcelas em aberto**:

| Quando | Horário | Etapa |
|---|---|---|
| 5 dias antes do vencimento | 07:00 | `d5` |
| 2 dias antes do vencimento | 07:00 | `d2` |
| No dia do vencimento | 07:00 | `d0` |
| No dia do vencimento | 16:00 | `d0_tarde` |

Antes de cada envio o servidor consulta o banco:

- se a parcela **já foi paga**, nada é enviado;
- cada etapa é **reservada uma única vez** (tabela `wpp_precobranca_envios`), então
  ninguém recebe o mesmo aviso duas vezes — nem se o serviço reiniciar no meio;
- se o envio falhar (WhatsApp desconectado, por exemplo), a etapa é liberada e ele
  tenta de novo mais tarde no mesmo dia.

Os textos de cada etapa, o liga/desliga por etapa e o botão **Enviar agora**
ficam no painel, em **Robô do WhatsApp → Pré-cobrança**.

**Promessa de pagamento:** se o cliente venceu dia 20 mas prometeu pagar dia
30, a pré-cobrança passa a usar a data prometida — ele recebe os mesmos avisos
(5 dias antes, 2 dias antes, no dia às 07:00 e às 16:00) contados a partir do
dia 30, e não mais do vencimento antigo. A data prometida é a mais recente
registrada na parcela.

> O lembrete antigo das **11:00** foi **desativado**. Quem manda os lembretes
> agora é a pré-cobrança (07:00 e 16:00).

**O que fazer:** publique esta pasta de novo e confira na aba **Conexão** se
aparece **Versão 5.8.0**. Sem QR Code — a sessão continua a mesma.

---

## ✅ v5.7.0 — aviso de pagamento para o administrador

Toda vez que uma parcela é confirmada como paga:

1. o **cliente** recebe a mensagem de pagamento concluído (texto da aba
   **Confirmação**);
2. **você** recebe um aviso no WhatsApp com quem pagou, qual parcela e qual
   valor.

Exemplo do aviso:

```
💰 Pagamento recebido!

👤 Cliente: Maria Silva
📄 Parcela: 3 de Carnê Kairós
💵 Valor: R$ 150,00
📅 Data do pagamento: 25/09/2026
📱 WhatsApp do cliente: (11) 99999-8888
```

O número que recebe o aviso, o texto e o liga/desliga ficam no painel, em
**Confirmação → Aviso para você (administrador)**. Tem um botão **Enviar teste**
para conferir na hora, sem esperar alguém pagar.

O aviso sai mesmo se o cliente não puder receber (número sem WhatsApp, por
exemplo): o dinheiro entrou e você precisa saber. Como cada parcela é reservada
antes do envio, o aviso nunca se repete.

**O que fazer:** publique esta pasta de novo e confira na aba **Conexão** se
aparece **Versão 5.7.0** ou superior. Sem QR Code — a sessão continua a mesma.

---

## ⚠️ v5.5.0 — atualização do motor (obrigatória se o cliente vê "Aguardando mensagem")

Se o painel diz **enviado** mas o cliente continua vendo o balão cinza
*"Aguardando mensagem. Essa ação pode levar alguns instantes"* — mesmo depois de
mandar outra mensagem — o problema está no motor, não no texto.

O motor antigo (Baileys 6.7) não entende o novo identificador **LID** do
WhatsApp. Quando o contato já foi migrado para essa identidade interna, o
pacote sai cifrado com uma chave que o celular dele não consegue abrir. A
mensagem é aceita, o painel marca *enviado* e o celular fica travado no balão.

Nesta versão:
- motor trocado para **Baileys 7.0.0** (tratamento correto de LID);
- o envio vai sempre pelo **número de telefone** (`@s.whatsapp.net`), nunca pelo `@lid`;
- quando chega mensagem em `@lid`, o servidor converte para o número real antes de responder;
- as chaves de conversa antigas são renovadas sozinhas na primeira subida.

**O que fazer:** publique esta pasta de novo (o build roda `npm install` e pega
a versão 7 sozinho). **Não precisa escanear QR Code** — a sessão do número
continua a mesma. Depois de ficar *Live*, abra a aba **Conexão** e confirme se
aparece **Versão 5.5.0**. Se algum cliente ainda estiver preso no balão, clique
em **Renovar chaves de conversa** e mande uma mensagem nova.

Este é o "motor" que mantém o WhatsApp da empresa conectado 24 horas por dia.
Ele recebe as mensagens dos clientes (só registra no painel, **não responde**),
envia as respostas escritas pelos atendentes e dispara a pré-cobrança, as
confirmações de pagamento e o aviso ao administrador.

> ⚠️ **Ele precisa ficar ligado o tempo todo e precisa de um disco próprio.**
> Hospedagens que "dormem" por inatividade ou que apagam o disco a cada deploy
> (Vercel, Netlify, Cloudflare, funções serverless) **não servem** — a sessão do
> WhatsApp morre e você precisa escanear o QR Code de novo toda hora.

---

## Novidade da v5.3 — nenhuma variável para preencher

Antes era preciso preencher 5 variáveis na hospedagem, e bastava errar uma para
o deploy "virar" mas nada funcionar. Agora o servidor se vira sozinho **e não usa
nenhuma chave secreta**:

| Antes você digitava | Agora |
|---|---|
| `WPP_API_KEY` | já vem com padrão que combina com o painel |
| `WPP_AGENT_SECRET` | já vem com padrão que combina com a IA |
| `SUPABASE_URL` | já vem apontando para o Backend certo |
| `SUPABASE_ANON_KEY` | **não é mais usada** |
| `WPP_AUTH_DIR` | **o servidor encontra o disco sozinho** |
| `WPP_PUBLIC_URL` | **detectada sozinha** |
| `SUPABASE_SERVICE_KEY` | **não é mais necessária** |

**Por que a chave de serviço saiu:** ela é a chave mestra do banco e o Backend
não a expõe (nem deveria). Em vez de copiar uma chave secreta para o painel da
hospedagem, o servidor pede as consultas para a função `wpp-agente`, que já roda
dentro do Backend com permissão total e é autenticada pela chave do agente. Isso
deixa o servidor **mais seguro** e com a **mesma permissão** de antes.

Ou seja: crie o serviço, espere ficar Live e pronto. Se algo falhar, abra
`SUA_URL/api/diagnostico` e ele diz em português o que está faltando.

---

## 1. Qual hospedagem usar

| Plataforma | Custo mensal | Dorme? | Disco | Veredito |
|---|---|---|---|---|
| **Render (plano Starter)** | ~US$ 7,25 (≈ R$ 39) | Não | Sim (US$ 0,25/GB) | **Padrão do projeto.** Só cliques, sem terminal |
| Railway | US$ 5 (≈ R$ 27) | Não | Sim | Funciona igual, um pouco mais barato |
| VPS (Hetzner/Contabo) | €4–5 (≈ R$ 25–30) | Não | Sim | Mais barato no longo prazo, exige terminal |
| Oracle Cloud Free | R$ 0 | Não | Sim | Existe plano gratuito, cadastro chato e máquina lenta |
| Fly.io | ~US$ 3,84 (≈ R$ 21) | Não | Sim | Foi a origem dos erros de porta. **Não recomendado** |

**Escolha do projeto: Render, plano Starter.** Cumpre os três requisitos de
produção (processo sempre vivo, disco persistente, reinício automático) e é o
mais difícil de errar.

Sobre "conexão boa": o que pesa não é o país do servidor, e sim ele nunca dormir,
ter disco e ficar perto do Backend. Use a região **virginia** (EUA, costa leste),
a mais próxima do Brasil (~120 ms). A conversa com o WhatsApp é feita pelos
servidores do WhatsApp, não pelo Brasil.

Custos extras possíveis: disco de 1 GB (US$ 0,25 — a sessão ocupa poucos MB) e
tráfego de saída (volume ínfimo).

---

## 2. Publicar no Render (caminho padrão, ~10 minutos)

Tudo pelo navegador, sem terminal. Uma vez só.

1. **GitHub.** Suba esta pasta em um repositório novo (pode ser privado).
   `package.json`, `Dockerfile` e `render.yaml` precisam ficar na **raiz** do
   repositório, não dentro de uma subpasta.
2. **Criar o serviço.** render.com → **New** → **Blueprint** → conecte o GitHub
   → escolha o repositório. O Render lê o `render.yaml` e já monta tudo: imagem
   Docker, plano, disco e checagem de saúde.
3. **Revisar.** Confirme na tela de revisão:
   - plano **Starter** (o Free **não** serve)
   - disco montado em **`/var/data`**, 1 GB
   - região **virginia**
4. **Variáveis.** Não precisa preencher nenhuma: o `render.yaml` já traz
   `TZ=America/Sao_Paulo` e o resto vem do código.
   **Não cadastre `PORT`**, não precisa de `SUPABASE_SERVICE_KEY` e nem de
   `WPP_AUTH_DIR`.
5. **Deploy.** O primeiro build leva de 3 a 8 minutos (instala o Baileys e
   compila dependências). Espere o status virar **Live**.
6. **Endereço.** Aparece no topo do serviço: `https://seu-servico.onrender.com`.
7. **Confira.** Abra `https://seu-servico.onrender.com/api/diagnostico`. Você
   deve ver `"ok": true`, `"pasta_persistente": true` e `"permissoes_banco":
   "completa"`. Depois cole só o endereço na aba **Conexão** do painel.

> Por que Starter e não Free: no Free o serviço hiberna após 15 min sem acesso e
> não tem disco. Hibernou = sessão caiu = QR Code de novo. Não existe contorno.

---

## 2.1 Alternativa: Railway (também funciona bem)

1. railway.app → **New Project** → **Deploy from GitHub repo** → escolha o
   repositório. Ele encontra o `Dockerfile` sozinho.
2. **Settings → Region:** `us-east4 (Virginia)`.
3. **Settings → Volumes → Add Volume:** montagem em **`/data`**.
4. **Variables:** nenhuma obrigatória. Não cadastre `PORT` nem `SUPABASE_SERVICE_KEY`.
5. **Settings → Networking → Generate Domain.**
6. Abra `/api/diagnostico` e confirme `"ok": true`.

Diferença única: aqui o disco é `/data`; no Render é `/var/data`. Como o servidor
procura os dois sozinho, você não precisa configurar nada.

---

## 3. Publicar em VPS próprio

```bash
# na VPS, com Docker instalado
docker compose up -d --build
```

O `docker-compose.yml` já cria o volume e reinicia o serviço sozinho (a pasta da
sessão é detectada automaticamente). Para HTTPS, coloque um proxy (Caddy ou
Nginx) na frente apontando para a porta 8080.

---

## 4. Variáveis de ambiente

| Variável | Obrigatória | O que é |
|---|---|---|
| `TZ` | recomendada | `America/Sao_Paulo` para os horários saírem certos. |
| `PORT` | não | Porta HTTP. Deixe em branco — a hospedagem injeta a sua. O servidor também escuta em 3000 e 3001 por segurança. |
| `WPP_API_KEY` | não | Senha que o painel usa para falar com este servidor. Tem padrão compatível. |
| `WPP_AGENT_SECRET` | não | Chave da função `wpp-agente` (IA). Tem padrão compatível. |
| `SUPABASE_URL` | não | Endereço do Backend. Tem padrão correto. |
| `WPP_AUTH_DIR` | não | Pasta da sessão. **Detectada sozinha.** Só use para forçar um caminho. |
| `WPP_PUBLIC_URL` | não | URL pública. Detectada sozinha no Render/Railway/Fly. |

Não existe mais nenhuma chave secreta para colar no painel da hospedagem: o
acesso ao banco é feito pela função `wpp-agente` dentro do Backend.

---

## 5. Como saber se está tudo bem

- `SUA_URL/api/diagnostico` → diz em português o que está faltando, se algo
  estiver. Este é o primeiro lugar para olhar quando algo não funciona.
- `SUA_URL/api/health` → resposta rápida usada pela hospedagem (deve ter
  `"ok":true`).
- Na aba **Conexão** do painel, o diagnóstico mostra versão, tempo no ar, sessão
  salva, contagem de mensagens e a data da última cobrança.
- Configure um monitor gratuito (UptimeRobot, BetterStack) apontando para
  `/api/health` a cada 5 minutos, com alerta por e-mail.
- Diagnóstico do servidor no painel mostra também `last_precobranca_manha` e
  `last_precobranca_tarde` (os últimos dias em que cada turno rodou).

O que o servidor já garante sozinho:
- **Sem mensagem duplicada**: cada parcela é "reservada" no banco antes do envio.
- **Pré-cobrança não se perde**: se o serviço estiver fora do ar às 07:00 ou às
  16:00, ele envia assim que voltar (e só uma vez por etapa, mesmo reiniciando
  várias vezes).
- **Reconexão automática** com espera progressiva.
- **Fila do atendente humano** entregue em até 5 segundos.
- **Mensagem não fica "Aguardando mensagem"**: duas proteções juntas. (1) as
  mensagens enviadas ficam guardadas por 30 minutos, então quando o celular do
  cliente pede o conteúdo de volta o servidor reenvia a original; (2) as chaves
  de conversa são renovadas automaticamente na primeira subida desta versão e,
  se voltar a acontecer, o botão **Renovar chaves de conversa** (aba Conexão)
  apaga só essas chaves e reconecta em segundos — **sem escanear QR Code**.
- **Desligamento limpo**: em deploy/reinício ele fecha a sessão antes de morrer,
  evitando arquivo de credencial corrompido.

---

## 6. Backup e volta atrás

**Backup da sessão (opcional):** no Render, o disco pode ser baixado em
**Disks → seu disco**. No Railway, em **Settings → Volumes → Download**. No VPS,
basta copiar a pasta da sessão. Se perder a sessão, escaneie o QR Code
novamente — nenhum dado de cliente é perdido, porque tudo fica no Backend.

**Voltar para a hospedagem anterior:** este servidor não depende de nenhuma
plataforma específica. Publique esta mesma pasta no host antigo e cole a URL nova
na aba **Conexão**. Nada muda no código.

**Nunca apague o serviço antigo antes de o novo responder** em `/api/health`
e de o botão **Enviar teste** chegar no seu celular.

---

## 7. Problemas comuns

| Sintoma | Causa provável | Solução |
|---|---|---|
| Build falha no Render | `Dockerfile` fora da raiz ou build sem memória | Veja **Logs**; confira a raiz do repositório |
| "Live" mas a URL não responde | Disco não montado em `/var/data` | Confira em **Disks** e abra `/api/diagnostico` |
| Conecta e cai em minutos | Sem disco (sessão não persiste) | `/api/diagnostico` aponta `pasta_persistente: false` — crie o disco |
| Serviço "dormiu" | Plano Free | Mude para **Starter** |
| QR não aparece | Serviço ainda subindo | Espere o deploy ficar Live e clique em Conectar |
| Conectado mas não responde cliente | **Comportamento normal** — o robô não responde clientes por projeto. As mensagens aparecem na aba **Conversas** | Nada a fazer. Responda manualmente pelo painel se quiser |
| Pré-cobrança não saiu hoje | Fora do horário, servidor antigo ou WhatsApp desconectado | Confira a versão (precisa ser 5.9.0) e use **Enviar agora** na aba Pré-cobrança |
| Pré-cobrança "demora" para terminar | Comportamento normal: cada cliente espera de 5 a 20 segundos | 10 clientes podem levar de 1 a 3 minutos. É de propósito, para não parecer disparo em massa |
| Cliente vê o balão cinza "Aguardando mensagem" | Envelope não decifrado. Na **v5.4.0 ou anterior** era o motor (Baileys 6.7 + LID): atualize para a **v5.5.0**. Já na 5.5.0, é a chave daquela conversa fora de sincronia (cliente trocou de aparelho ou reinício no meio da troca) | Atualize para a v5.5.0 e depois, na aba **Conexão**, clique em **Renovar chaves de conversa**. Isso apaga só as chaves de conversa (`session-*.json`), reconecta em segundos e **não pede QR Code** |

---

## 8. Portas (só se algum dia der "Request failed")

O servidor escuta em **8080, 3000 e 3001** ao mesmo tempo, e usa
`process.env.PORT` quando a plataforma injeta uma porta. Isso cobre Render,
Railway, Fly e VPS sem nenhuma configuração. Se a URL ficar muda com o serviço
ligado, confira se alguém cadastrou `PORT` manualmente com um valor errado e
remova essa variável.