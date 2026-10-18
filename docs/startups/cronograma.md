# Cronograma de entregas — JobSpark na Fase III

> Atualizado em 18/10/2026. Datas do Edital nº 03/2026 (itens 1.2, 5 e 6). O edital avisa que as datas podem mudar por retificação: conferir os canais oficiais todo mês.

## 1. Datas-chave

| Quando | O quê |
|---|---|
| até **28/11/2026, 23h59** | Inscrição e envio da proposta |
| 30/11 – 11/12/2026 | Análise de mérito e elegibilidade |
| 14 – 18/12/2026 | Resultado preliminar |
| 21 – 25/12/2026 | Prazo de recurso |
| 28 – 31/12/2026 | Resultado final e homologação das 20 startups |
| até 31/12/2026 | Confirmação da participação (item 6.2) |
| fev/2027 | Início da Fase III · Sprint 1 |
| ago/2027 | Sprint 7 · **Demo Day**, formatura e relatório final (item 5.3) |

Regra que elimina (item 5.5): presença mínima de **60%** nos encontros on-line. Uma falta só é abonada com justificativa por escrito em até 48 h.

## 2. Até a inscrição (out – nov/2026)

- [x] MVP em produção com segurança, RH, LGPD e planos
- [x] Proposta (Anexo I) revisada: só afirma o que está no ar
- [ ] Enviar o formulário com folga (meta: até 21/11)
- [ ] Roteiro de demonstração de 3 minutos: vaga → QR → entrevista por voz → dossiê → ficha do gestor → contratar → admissão
- [ ] 1 a 3 empresas piloto no Ceará, com vaga real, registrando candidatos, tempo economizado e opinião do dono

## 3. Resultado e preparação (dez/2026 – jan/2027)

- [ ] Ler o resultado preliminar e a nota por critério; recurso com provas, se preciso
- [ ] Confirmar a participação até 31/12
- [ ] Linha de base em janeiro: vagas, candidatos, entrevistas concluídas e tempo até contratar (o painel `/empresa/rh` já calcula)
- [ ] Banco de dados em plano pago antes de receber dados de clientes reais

## 4. Sprint por sprint (fev – ago/2027)

| Mês | Sprint do edital | Entrega do JobSpark | Produto | Meta acumulada |
|---|---|---|---|---|
| fev/27 | 1. Modelo de Negócios | Lean Canvas e Business Model Canvas; proposta de valor por segmento; quadro de concorrentes; 15 entrevistas com donos de PME | Onboarding da empresa sem ajuda (cadastro → 1ª vaga em 5 min) | 3 pilotos · 50 candidatos |
| mar/27 | 2. Análise de Mercado | TAM/SAM/SOM com RAIS, IBGE e Sebrae; segmentos e canais; mapa de PMEs do Ceará | Painel de uso por empresa no super admin | 5 pilotos · 120 candidatos |
| abr/27 | 3. Tecnologia e Escalabilidade | Relatório da IA com dados reais: kappa da IA × ficha do gestor; custo de IA por entrevista | Retreinar os modelos com dados reais; várias pessoas por empresa; verificação de antecedentes com fornecedor real ou removida; alertas de erro | 8 pilotos · 250 candidatos · kappa medido |
| mai/27 | 4. Marketing e Vendas | Funil de vendas; 2 estudos de caso; parcerias com associações comerciais e contadores; primeiros planos pagos | Página de preços com contato comercial; cartaz com QR para imprimir | 10 pilotos · 30 vagas · 2 pagantes |
| jun/27 | 5. Legal e PI | Contrato SaaS e acordo de tratamento de dados (LGPD); termos revisados na mentoria jurídica; pedido da marca no INPI; forma de faturar os pilotos pagos | Exportação dos dados do candidato (portabilidade) | 3 pagantes |
| jul/27 | 6. Finanças e Gestão | Plano financeiro de 18 meses; MRR, CAC, churn, custo de IA por entrevista, margem por plano | Relatório mensal automático para a empresa cliente | 5 pagantes · MRR ≥ R$ 1.400 · 500 candidatos |
| ago/27 | 7. Captação | Pitch deck; inscrição em Centelha, Finep ou anjo; **Demo Day**; relatório final | Congelar mudanças grandes 2 semanas antes do Demo Day | Metas fechadas · certificado |

## 5. Rotina de todo mês

- [ ] Presença ≥ 60% e justificativa escrita de faltas em 48 h
- [ ] Relatório mensal da residência
- [ ] Atualizar a tabela de metas (seção 6) com os números do `/empresa/rh` e do super admin
- [ ] No máximo um deploy grande por mês, com ensaio e registro do deploy
- [ ] Emitir as faturas das empresas em plano pago (cobrança manual)

## 6. Metas mês a mês (preencher no fim de cada mês)

| Mês | Pilotos | Vagas | Candidatos | Pagantes | MRR (R$) | Kappa | Presença |
|---|---|---|---|---|---|---|---|
| jan/27 (base) | | | | | | – | – |
| fev/27 | | | | | | – | |
| mar/27 | | | | | | – | |
| abr/27 | | | | | | | |
| mai/27 | | | | | | | |
| jun/27 | | | | | | | |
| jul/27 | | | | | | | |
| ago/27 | | | | | | | |

## 7. Como entra uma empresa nova

1. A empresa se cadastra sozinha como Empresa (e-mail, senha e CNPJ, ou login com Google).
2. Já começa no **Starter** (1 vaga ativa, 20 entrevistas por mês) e publica a primeira vaga na hora.
3. No painel de vagas ela vê o uso. No limite, recebe um aviso e o link "Trocar de plano".
4. Combinado o plano, sai a fatura. Quando o pagamento entra: **super admin → Empresas → plano** (Pro, Scale ou Enterprise). Vale na hora.
5. Sem pagamento no mês seguinte, o plano volta para Starter: as vagas continuam no ar, mas não dá para publicar nova acima do limite.

Ainda não existe: aviso automático de cadastro novo, cobrança automática e várias pessoas na mesma empresa (planejado para abr/27).

## 8. Riscos

| Risco | Sinal | Resposta |
|---|---|---|
| Não ser selecionado | Resultado preliminar baixo | Recurso com provas de piloto; seguir com os pilotos |
| Poucos pilotos | Menos de 3 em fev/27 | 3 meses grátis em troca de depoimento; rede SiDi/Softex |
| Custo de IA alto | Mais de R$ 1 por entrevista | Limites de plano; modelo mais barato ou entrevista mais curta |
| Incidente no banco | Perda de dados | Plano pago com restauração antes de fev/27 |
| Faltas | Presença abaixo de 70% | Agenda bloqueada; justificar em 48 h |
