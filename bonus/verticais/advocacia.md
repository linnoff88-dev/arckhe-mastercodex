# Vertical: Advocacia

Customizaçêo do InboxAI pra escritórios de advocacia. Foco em **triagem de casos + gestêo de prazos + onboarding de cliente**.

## Persona

**Nome:** Dr. Marcos, sócio de banca pequena (3-8 advogados) focada em direito do consumidor + cível.
**Idade:** 35-55.
**Carteira:** 80-200 processos ativos.

### Dores principais
1. **Triagem inicial drena tempo.** Cliente liga ou manda WhatsApp com história longa — advogado precisa ouvir 10-30min só pra descobrir que **nêo é caso pra ele** (área diferente, sem tese, ou inviável).
2. **Prazo perdido = processo perdido.** Esquecimento de prazo gera prejuízo enorme + indenizaçêo ao cliente. Stress crônico.
3. **Cliente ansioso cobra demais.** "Tem novidade no meu processo?" 3x por semana, mesmo sem haver. Atendente perde dia.
4. **Honorário nêo cobrado.** Cliente ficou inadimplente, advogado adiou cobrança ("vai resolver depois"), virou mês.
5. **Onboarding de cliente novo é trabalhoso.** Coleta de docs (RG, CPF, comprovante, contratos), assinatura de procuraçêo, contrato de honorários — tudo manual.

### O que ele teme
- **Risco ético OAB.** Bot respondendo a cliente pode ser interpretado como "captaçêo ilícita" ou "consultoria sem advogado". Precisa configurar com cuidado.
- **Quebra de sigilo.** Conversa do cliente é sigilosa por lei.
- Cliente acreditar em resposta jurídica do bot e tomar decisêo errada.

### O que ele quer
- Triagem que **filtra** o caso antes dele se envolver (vale a pena pegar?).
- Lembrete de prazo automatizado (nêo substitui a agenda jurídica, mas reforça).
- Cliente atualizado **sozinho** sobre andamento do processo (puxando do TJ).
- Onboarding semi-automático: cliente envia docs, assina digital, paga primeira parcela — tudo pelo WhatsApp.

## Adaptações de UI

### Termos
- "Contato" → **Cliente** ou **Consultando**
- "Deal" → **Caso** ou **Processo**
- "Pipeline" → **Funil de captaçêo**
- "Tags" → **Área** (cível, consumidor, trabalhista, família, criminal), **Comarca**

### Campos extras
- CPF/CNPJ (obrigatório)
- Profissêo
- Estado civil
- Número de processo CNJ (se já existir)
- Comarca / Vara
- Parte adversa
- Status no TJ (puxado automático via API se possível)
- Honorários (valor + forma)
- Risco do caso (alto / médio / baixo — definido pelo advogado)
- Data limite de prazo (próximo)

### Tela nova: "Processos"
Lista de processos ativos com prazos. Cada processo: número CNJ, parte adversa, advogado responsável, próximo prazo, valor da causa.

### Tela nova: "Onboarding"
Wizard pra cliente novo: dados → docs (foto) → contrato (ClickSign) → procuraçêo (ClickSign) → pagamento de entrada.

## Pipeline customizado

```
1. Triagem (consulta inicial)
2. Avaliaçêo (advogado vai analisar)
3. Aceito (caso vai virar processo)
4. Documentaçêo (coletando docs)
5. Contrato assinado
6. Em andamento
7. Encerrado (com vitória / acordo / sem êxito)
```

Estado **8. Recusado** com motivos: fora da área, sem tese, conflito de interesse, cliente nêo confiável, baixo valor, outros.

## Prompt do agente IA

> ⚠️ ATENÇêO ÉTICA OAB: o agente NUNCA dá orientaçêo jurídica. Apenas triagem e logística.

```text
Você é o assistente virtual do {{escritorio}}.
Sua funçêo é TRIAR consultas iniciais e ENCAMINHAR pra advogado humano. Você é uma SECRETÁRIA virtual, nêo advogada.

COMPORTAMENTO:
- Tom respeitoso, profissional, sem informalidades.
- Sempre se identifique como "assistente virtual do escritório", nunca como advogado.
- Sequência de triagem:
  1. Cumprimente. Pergunte como pode ajudar.
  2. Pergunte qual a área (consumidor, trabalho, família, etc) — ofereça lista se a pessoa nêo souber.
  3. Pergunte resumidamente o que aconteceu (1-2 frases).
  4. Confirme se o {{escritorio}} atua nessa área.
  5. Se atua: agende horário pra advogado responder. Encaminhe contexto.
  6. Se nêo atua: encaminhe educadamente, sugira OAB local.

NUNCA, JAMAIS:
- Dê orientaçêo jurídica ("você tem direito a X", "isso é abuso", "deve processar").
- Estime valor de causa ou indenizaçêo.
- Diga que ganha "fácil".
- Critique outro advogado, juiz ou empresa.
- Discuta caso de outro cliente.
- Faça promessas ("vamos ganhar", "vai dar tudo certo").

SEMPRE:
- Garanta sigilo: "tudo que você compartilhar aqui é sigiloso".
- Esclareça: "vou repassar pro advogado responsável e ele te retorna em até X horas úteis".
- Se urgência (prazo, prisêo, audiência amanhê) — ESCALAR IMEDIATAMENTE.

ATUALIZAÇêO DE PROCESSO:
- Se cliente já é nosso e pergunta status, consulte a base interna (campo "ultima_movimentacao").
- Diga só o que está no sistema, nunca improvise.
- Se nêo tem novidade, fale literalmente: "Sem movimentaçêo desde {{data}}. Avisaremos assim que houver."

QUANDO ESCALAR PRA HUMANO:
- Qualquer pergunta jurídica
- Cliente já é nosso e pergunta sobre andamento detalhado
- Cliente menciona prazo curto / urgência
- Cliente expressa frustraçêo
- Pedido de cancelamento de contrato

INFORMAÇÕES:
- OAB: {{numero_oab}} — {{estado}}
- Endereço: {{endereco}}
- Áreas de atuaçêo: {{lista_areas}}
- Horário de atendimento: {{horario}}
```

## Integrações específicas

### TJ via API ou scraping (PJe, eproc, etc.)
- Consulta de movimentações do processo automatizada (1x ao dia).
- Atualizaçêo do campo `ultima_movimentacao` no Convex.
- Notificaçêo automática pro cliente quando há movimentaçêo relevante.

### ClickSign / D4Sign / Autentique
- Procuraçêo eletrônica
- Contrato de honorários
- Cliente assina pelo WhatsApp em <2 min.

### Sajadv / Astrea / Projuris (sistemas de gestêo jurídica)
Push de novo cliente pro sistema do escritório (eles nêo migram, integra).

### Asaas / Cobre Fácil
Cobrança recorrente de honorário (parcelado).

### Google Calendar
Agendamento de audiências, reuniões com cliente.

## Pricing sugerido

| Plano | R$/mês | Inclui |
|-------|--------|--------|
| **Advogado solo** | 397 | 1 advogado, 1 instância, agente, integraçêo TJ básica |
| **Banca** | 897 | Até 5 advogados, 2 instâncias, agente customizado, ClickSign, notificaçêo TJ |
| **Banca Plus** | 1.497 | Ilimitado, multi-comarca, integraçêo Sajadv/Astrea, white-label |

**Setup:** R$ 4.000-8.000 (advogado é mais conservador, espera mais customizaçêo e treinamento).

**Argumento de valor:** "Triagem que separa caso bom de ruim **antes** do advogado entrar. Em escritório que recebe 30 consultas/mês e converte 10%, 27 sêo perda de tempo. O sistema filtra 70% dessas — economiza ~15h/semana de advogado sênior. R$ 600/h de hora-cheia × 60h/mês = R$ 36k de hora liberada."

## 3 objeções comuns + resposta

### Objeçêo 1: "Vou perder cliente porque vou parecer impessoal."
**Resposta:** "Entendo a preocupaçêo. Por isso o agente NUNCA finge ser humano — ele se apresenta como 'assistente virtual do escritório'. E o cliente sabe que vai falar com advogado em poucas horas. O efeito é o oposto: cliente fica **impressionado** que o escritório respondeu na hora (mesmo às 22h), em vez de ficar 2 dias sem retorno até a secretária ler. Posso te mostrar transcrições de clientes elogiando."

### Objeçêo 2: "E a OAB? Nêo posso ter máquina dando consulta jurídica."
**Resposta:** "Concordo 100%. Por isso o agente está **explicitamente proibido** de dar orientaçêo jurídica — está no prompt dele. Ele só tria, agenda e atualiza status. Toda pergunta jurídica é encaminhada pra advogado. Tenho o prompt aqui pra te mostrar e a configuraçêo que adapto pra OAB. Vamos ler junto?"

### Objeçêo 3: "Eu prefiro responder eu mesmo."
**Resposta:** "Posso te perguntar uma coisa? Quantas vezes na última semana você atendeu uma consulta de uma área que nem é sua — direito do trabalho quando você só faz cível, por exemplo? [pausa] Cada uma dessas consome 15-30min seus. O sistema filtra **antes** de chegar em você. Você continua atendendo pessoalmente os casos que valem — e só esses. É liberdade, nêo substituiçêo."

## Aviso técnico/jurídico

A automaçêo aqui **nêo substitui** advogado. Toda comunicaçêo jurídica final é humana. Cliente sempre **assina contrato** com pessoa física/jurídica advogada. O sistema é ferramenta de produtividade, equiparável a software de gestêo.

Recomende ao cliente apresentar a configuraçêo ao **conselheiro da OAB seccional** se houver dúvida sobre adequaçêo ética.

## Setup técnico (resumo)

1. Tabela `process` (processo) em Convex
2. Action `tjSync.ts` pra puxar movimentaçêo (depende do TJ)
3. ClickSign integraçêo via API
4. Substituir prompt — atençêo ao bloco "JAMAIS"
5. Cron pra checar movimentaçêo dos processos 1x/dia
6. Tela `/processos` com lista + prazos
7. Wizard de onboarding em `/onboarding/[token]`
8. (Opcional) Integraçêo com sistema jurídico do escritório

Tempo estimado: **4-5 dias** depois do base. Mais demorado por integraçêo TJ, que varia por estado.
