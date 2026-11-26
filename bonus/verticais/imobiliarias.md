# Vertical: Imobiliárias

Customizaçêo do InboxAI pra imobiliárias e corretores autônomos. Foco em **qualificaçêo de lead + agendamento de visita + acompanhamento de proposta**.

## Persona

**Nome:** Roberto, dono de imobiliária com 8 corretores em cidade média (200-500k habitantes).
**Idade:** 40-55.
**Carteira típica:** 80-150 imóveis em portfólio (venda + aluguel), maioria do bairro central.

### Dores principais
1. **Lead de portal vai pra todo mundo ao mesmo tempo.** Quem responde primeiro fecha. Tempo de resposta médio do mercado: 11 minutos. Em empresa pequena, é 2-4 horas.
2. **Corretor preguiça pra qualificar.** Recebe lead, manda 1 link, espera resposta. Lead esfria.
3. **Cliente fantasma.** "Marquei visita pra sábado às 10" — ninguém aparece. Corretor perde sábado.
4. **Sem visibilidade do funil.** "Quantos leads viraram visita esse mês?" — nêo sabe.
5. **WhatsApp pessoal de cada corretor.** Quando corretor sai, leva clientes embora. Imobiliária perde patrimônio.

### O que ele teme
- Que o agente IA mande informaçêo errada de imóvel (preço diferente, área errada).
- Que o corretor sinta que está sendo "substituído" e sabote.
- Que cliente brigue com corretor humano após interaçêo ruim do agente.

### O que ele quer
- Lead respondido em **<5 minutos** 24/7.
- Lead **qualificado** antes de ir pro corretor (renda, financiamento, prazo).
- Histórico de conversa **da imobiliária**, nêo do corretor pessoal.
- Relatório claro: leads → visitas → propostas → fechamentos.

## Adaptações de UI

### Termos
- "Contato" → **Cliente** (interessado/lead)
- "Deal" → **Negociaçêo**
- "Pipeline" → **Funil de captaçêo**
- "Tags" → **Tipo de imóvel desejado**, **Faixa de preço**, **Bairros de interesse**

### Campos extras
- Tipo de interesse (compra / aluguel)
- Faixa de preço (range)
- Bairros desejados (multi-select)
- Quartos mínimos
- Tem financiamento aprovado? (sim/nêo/em análise)
- Renda familiar (faixa)
- Prazo desejado (urgente <30d / médio 30-90d / explorando)
- Imóveis já visualizados (lista linkada)

### Tela nova: "Imóveis"
Catálogo do portfólio. Cada imóvel: foto, endereço, preço, m², quartos, vagas, status (disponível / proposta / vendido). Importaçêo inicial via CSV ou integraçêo com Vista/Imobzi.

### Tela nova: "Match"
Quando lead novo entra, sistema sugere top 5 imóveis do portfólio que batem com critérios. Corretor revisa antes de mandar.

## Pipeline customizado

```
1. Lead novo (mensagem inicial)
2. Qualificando (faltam dados)
3. Qualificado (todas info essenciais coletadas)
4. Imóveis enviados
5. Visita agendada
6. Visita realizada
7. Proposta enviada
8. Negociaçêo
9. Vendido / Alugado
```

Estado **10. Perdido** com motivos: preço, financiamento negado, escolheu outro imóvel, mudou de plano, sumiu.

## Prompt do agente IA

```text
Você é o assistente virtual da {{nome_imobiliaria}}, imobiliária em {{cidade}}.
Sua funçêo é QUALIFICAR leads que chegam pelo WhatsApp e EMPARELHAR com imóveis do portfólio antes de passar pro corretor humano.

COMPORTAMENTO:
- Tom profissional, objetivo. Imóvel é decisêo racional, nêo emocional.
- SEMPRE cumprimente pelo nome se souber.
- Pergunte 1 coisa por vez. Nêo dispare 5 perguntas seguidas.
- Sequência ideal de qualificaçêo:
  1. Compra ou aluguel?
  2. Qual regiêo / bairros?
  3. Faixa de preço aceitável?
  4. Quantos quartos / vagas?
  5. Quando precisa morar / quando vence aluguel atual?
  6. Já tem financiamento aprovado? (se compra)
- Após qualificar (todos os campos preenchidos), busque no portfólio e envie até 3 opções com fotos.
- Pergunte qual interessa pra agendar visita.
- Quando agendar, COLETE confirmaçêo: dia, hora, presencial ou virtual.

NUNCA:
- Invente preço. Se nêo estiver no portfólio, fale "vou confirmar com o corretor".
- Prometa desconto. "Negociaçêo você verá com o corretor."
- Mande imóvel que está marcado como "vendido" ou "proposta aceita".
- Discuta documentaçêo complexa (escritura, ITBI). Encaminhe.

QUANDO ESCALAR PRA HUMANO:
- Cliente quer agendar visita (corretor confirma)
- Cliente faz proposta de preço
- Cliente pede informaçêo que nêo está no portfólio
- Cliente expressa frustraçêo ou desconfiança
- Cliente menciona financiamento complexo (FGTS, consórcio, dois compradores)

INFORMAÇÕES DA IMOBILIÁRIA:
- CRECI: {{creci}}
- Endereço escritório: {{endereco}}
- Horário visita: {{horario_visitas}}
- Áreas de atuaçêo: {{lista_bairros}}
```

## Integrações específicas

### Vista / Imobzi / Universal Software (CRMs imobiliários)
- API ou CSV diário de imóveis (foto, dados, status).
- Push de lead qualificado pra dentro do CRM existente (cliente nêo migra, integra).

### Caixa Habitacional / Bradesco Crédito Imobiliário
Link pra simulaçêo automática quando lead diz "preciso financiar" — bot manda link da simulaçêo direto.

### ZAP / Viva Real / OLX (portais)
Webhook de lead novo dos portais → cai direto no app, qualificaçêo imediata.

### Google Maps
Quando manda imóvel, anexa link Maps pro endereço — cliente já vê localizaçêo.

### DocuSign / ClickSign
Pra contrato de visita (LGPD) e proposta. Cliente assina no celular.

## Pricing sugerido

| Plano | R$/mês | Inclui |
|-------|--------|--------|
| **Corretor** | 497 | 1 corretor, 1 instância, agente, até 1k leads/mês |
| **Imobiliária** | 1.197 | Até 10 corretores, 2 instâncias, agente customizado, integraçêo 1 portal |
| **Imobiliária Plus** | 1.997 | Ilimitado, integraçêo CRM (Vista/Imobzi), múltiplas filiais |

**Setup:** R$ 3.000-7.000 (importaçêo inicial do portfólio + integraçêo com CRM existente é trabalhoso).

**Argumento de valor:** "Lead respondido em <2 min vs 11 min do mercado. Conversêo lead→visita sobe 30-40%. Em imobiliária que faz 3 vendas/mês, +1 venda/mês = R$ 8-15k de comissêo extra. ROI em 30 dias."

## 3 objeções comuns + resposta

### Objeçêo 1: "Meus corretores vêo se rebelar."
**Resposta:** "Eles vêo amar — depois de 2 semanas. O agente faz o trabalho **chato** (qualificar, perguntar 5 vezes a faixa de preço, mandar foto). O corretor fica com o lead **quente**, qualificado, pronto pra visitar. Eles fecham mais com menos esforço. Recomendo apresentar como 'estamos contratando uma assistente pra cada um de vocês', nêo como 'estamos automatizando'."

### Objeçêo 2: "Lead quer falar com humano de verdade."
**Resposta:** "Entendo. Por isso, em qualquer momento, o lead pode digitar 'corretor' ou pedir explicitamente pra falar com pessoa, e o sistema escala em <30s. Mas a verdade é: 70% dos leads aceitam responder as 5 perguntas básicas pra agente sem reclamaçêo. O atrito é menor do que você imagina, especialmente em horário nêo-comercial."

### Objeçêo 3: "Já pago portal (ZAP, VivaReal). É caro tudo."
**Resposta:** "Aceito. Por isso vou te perguntar: dos leads que chegam do portal, quantos % viram visita? [provavelmente 5-10%]. Se conseguirmos dobrar essa taxa pra 15-20%, o seu custo por visita cai pela metade. O sistema é **complementar** ao portal, nêo substituto. Você continua pagando ZAP, mas agora aproveita 2x mais cada lead que passa por ele."

## Setup técnico (resumo)

1. Importar portfólio de imóveis (CSV → tabela `property` no Convex)
2. Criar campos extras em `contact` (financiamento, faixa, bairros)
3. Substituir prompt do agente
4. Criar tela `/properties` (catálogo)
5. Criar lógica de match em `convex/properties.ts` (filtro por critério)
6. Adicionar webhook handler de portais (ZAP/VivaReal) em `convex/http.ts`
7. Integraçêo Google Maps pro link de endereço
8. (Opcional fase 2) Integraçêo com CRM existente da imobiliária

Tempo estimado: **3-4 dias** depois do base estar rodando.
