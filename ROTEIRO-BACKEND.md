# Roteiro do back-end da pescAÍ (para o Claude Code)

> Abra o Claude Code dentro da pasta deste repositório e diga:
> **"Leia o ROTEIRO-BACKEND.md e construa o back-end da pescAÍ seguindo ele, etapa por etapa. Me pergunte antes de cada coisa que eu precise fazer numa conta minha."**

## O que é a pescAÍ

SaaS brasileiro de atendimento e vendas pelo WhatsApp com IA. Cada empresa cliente:
1. assina o plano (R$ 110/mês, cobrança recorrente no Asaas);
2. treina a própria IA com produtos, preços, perguntas frequentes, condições de pagamento/entrega e arquivos;
3. conecta o número de WhatsApp da empresa;
4. a IA responde os clientes dela, conduz até a venda e transfere para um humano quando precisar;
5. acompanha conversas, vendas e funil num painel.

## O que já existe neste repositório

- `index.html` — site de vendas (estático, GitHub Pages: https://agenciabrandview-cmd.github.io/pescai/).
- `app/index.html` — protótipo do painel (HTML/CSS/JS num arquivo só). Hoje salva tudo no `localStorage` e tem dados de exemplo. **Use as telas e o visual dele como referência**: login, painel, conversas, treinamento da IA (arquivos, produtos e preços, FAQ, pagamento e entrega, comportamento), testar a IA, perfil da empresa.
- Identidade: vermelho `#e3122d`, vinho `#5c0b19`, preto e branco; títulos em Nunito 900, texto em Manrope; nome escrito "pescAÍ".

## Stack escolhida

| Parte | Ferramenta |
|---|---|
| Login, banco de dados, arquivos | **Supabase** (Auth + Postgres + Storage) |
| Lógica do servidor | **Supabase Edge Functions** (TypeScript/Deno) |
| WhatsApp | **API oficial do WhatsApp (Cloud API da Meta)** — não usar APIs não oficiais por QR code (risco de banimento do número do cliente) |
| IA | API do Claude (Anthropic) |
| Pagamento | **Asaas** (assinatura recorrente + webhook) |
| Front-end | Continuar estático no GitHub Pages, chamando o Supabase pelo `supabase-js` |

## Etapas

### Etapa 1 — Supabase: contas e banco
- Projeto Supabase; Auth por e-mail e senha.
- Tabelas (todas com `empresa_id` e **Row Level Security**: cada empresa só vê os próprios dados):
  - `empresas` (nome, telefone, endereço, horários, logo, status_assinatura, asaas_customer_id, asaas_subscription_id)
  - `membros` (user_id → empresa_id, papel)
  - `produtos` (nome, categoria, preço, descrição)
  - `faq` (pergunta, resposta)
  - `condicoes` (texto de pagamento e entrega)
  - `comportamento_ia` (tom, regras, quando transferir para humano)
  - `arquivos` (Storage; texto extraído para a IA consultar)
  - `whatsapp_contas` (phone_number_id, waba_id, token criptografado)
  - `contatos` e `conversas` (status: ia, humano, venda, encerrada)
  - `mensagens` (direção, texto, horário, autor: cliente/ia/humano)
- Trocar o `localStorage` do `app/index.html` por leitura e escrita no Supabase, mantendo as mesmas telas.

### Etapa 2 — Treinamento e "Testar a IA" de verdade
- Edge Function `responder` que monta o contexto da empresa (perfil, produtos, FAQ, condições, comportamento, trechos dos arquivos) e chama a API do Claude.
- A IA nunca inventa preço ou produto que não está cadastrado; se não souber, oferece transferir para humano.
- Ligar a tela "Testar a IA" do painel nessa função.
- Leitura de PDF/Word/CSV/TXT enviados (extrair texto e guardar).

### Etapa 3 — WhatsApp oficial
- App no Meta for Developers com WhatsApp Cloud API.
- Edge Function `whatsapp-webhook`: valida a assinatura da Meta, recebe mensagens, grava em `mensagens`, chama `responder` e envia a resposta pela Cloud API.
- Respeitar a janela de 24 horas (fora dela, só modelos aprovados).
- Botão "Assumir conversa" no painel: pausa a IA naquela conversa e deixa o humano responder pelo painel.
- Onboarding do número do cliente: começar conectando manualmente (phone_number_id + token) para os clientes de teste; depois implementar o Embedded Signup da Meta (exige a conta virar Provedor de Tecnologia).

### Etapa 4 — Pagamento (Asaas)
- No cadastro: criar cliente e assinatura de R$ 110/mês no Asaas pela API e mostrar o link de pagamento.
- Edge Function `asaas-webhook`: pagamento confirmado → `status_assinatura = ativa`; atrasado/cancelado → bloquear o painel e pausar a IA (com aviso e alguns dias de tolerância).
- Começar no **sandbox** do Asaas.

### Etapa 5 — Painel com dados reais
- Indicadores, gráfico de conversas e vendas, funil e atividade recente lendo do banco (hoje são de exemplo).
- Marcar venda fechada (pela IA ou manualmente) com valor.

### Etapa 6 — Publicar e testar
- Variáveis secretas só no Supabase (nunca no front-end nem no GitHub): chave da Anthropic, token da Meta, chave do Asaas.
- Testar o fluxo completo com um número de teste: cadastro → pagamento sandbox → treino → mensagem no WhatsApp → resposta da IA → venda no painel.
- Depois, 5 a 10 empresas reais em teste antes de anunciar.

## O que o Antonio precisa fazer nas contas dele (o Claude Code pede na hora certa)

- Criar conta no **Supabase** e passar a URL e as chaves do projeto.
- Criar conta no **Meta for Developers**, o app do WhatsApp e um número de teste.
- Criar a chave da **API da Anthropic** (console.anthropic.com).
- Criar conta no **Asaas** (com CNPJ/CPF) e gerar a chave da API do sandbox.

## Regras

- Português do Brasil em todas as telas e mensagens.
- Nada de dados de exemplo misturados com dados reais.
- Nenhuma chave secreta no repositório.
- Um commit por etapa, com mensagem clara.
