# JobSpark — Documentação técnica (versão pública)

> Versão de 18/10/2026, alinhada ao que está em produção em [jobspark.com.br](https://jobspark.com.br).
> O código-fonte é privado. Esta versão descreve arquitetura, dados, fluxos, IA, segurança e qualidade sem detalhes internos de infraestrutura.

## Sumário

1. [Visão geral](#1-visão-geral)
2. [Arquitetura](#2-arquitetura)
3. [Stack](#3-stack)
4. [Backend](#4-backend)
5. [Modelo de dados](#5-modelo-de-dados)
6. [API](#6-api)
7. [Fluxos principais](#7-fluxos-principais)
8. [Inteligência artificial](#8-inteligência-artificial)
9. [Frontend](#9-frontend)
10. [Segurança e LGPD](#10-segurança-e-lgpd)
11. [Qualidade e testes](#11-qualidade-e-testes)
12. [Infraestrutura e deploy](#12-infraestrutura-e-deploy)
13. [Limitações conhecidas e próximos passos](#13-limitações-conhecidas-e-próximos-passos)

---

## 1. Visão geral

O JobSpark é um SaaS multiempresa de recrutamento para pequenas e médias empresas. A empresa publica a vaga com link e QR Code; a entrevistadora **Aria** (IA generativa) conduz a primeira entrevista por texto ou áudio; a empresa recebe um dossiê com a nota e a evidência de cada ponto, registra a decisão humana na ficha do gestor, contrata e acompanha a admissão.

| Papel | O que faz |
|---|---|
| Candidato | Entrevista pelo link/QR (sem login) ou pelo site (logado, inclusive com Google); perfil, candidaturas, exclusão dos próprios dados |
| Empresa | Vagas, pipeline, dossiê, ficha do gestor, convite por WhatsApp, banco de talentos, RH, uso do plano |
| Super admin | Empresas, planos e faturamento manual |

## 2. Arquitetura

<img src="img/infraestrutura.png" alt="Infraestrutura do JobSpark" width="100%" />

- **Cada empresa é um tenant isolado:** toda consulta filtra pela empresa do usuário logado, e a empresa nunca é aceita a partir do corpo da requisição.
- **Frontend sem estado de servidor:** renderiza a interface e conversa com a API REST.
- **Arquivos pessoais** (vídeo e currículo) ficam em armazenamento privado e só saem por rota que confere quem pode ver.
- **Serviço de ML separado**, chamado pela API.

## 3. Stack

| Camada | Tecnologia |
|---|---|
| Frontend | Next.js 15 (App Router), React 19, Tailwind CSS v4, Zustand |
| Backend | Bun, Hono 4, Prisma 6, JWT |
| Banco | PostgreSQL com pgvector (Neon), migrations versionadas |
| IA | Groq (`gpt-oss-120b`, Whisper large-v3), Google `gemini-embedding-001` |
| ML | Python, FastAPI, scikit-learn |
| Nuvem | Google Cloud Run, Cloud Build, Cloud Storage, Secret Manager |
| Testes | `bun test`, TypeScript (`tsc`) no frontend e no backend, Playwright, roteiros de ensaio |

## 4. Backend

Organizado em três camadas:
- **rotas:** HTTP, autenticação e o filtro da empresa escritos na própria rota, para a auditoria automática conferir;
- **regras puras:** planos, admissão, painel, entrevista, aderência, ficha do gestor, banco de talentos, testadas isoladamente;
- **serviços:** agente da Aria, score 360°, DISC e verificação de antecedentes (demonstração).

**Autenticação**
- Login por e-mail e senha (senha com hash) ou com Google (token validado no servidor).
- JWT em cookie `httpOnly`.
- Entrevista sem login: o candidato recebe um token opaco e o banco guarda só o hash.
- Convite de entrevista: código de 8 caracteres, guardado só como hash e válido por 7 dias.

**Autorização:** uma sessão de entrevista só é aberta pelo próprio candidato (token), pela empresa dona da vaga ou pelo super admin. Para quem não pode ver, a resposta é "não encontrado", sem revelar que o registro existe.

**Proteções:** limite de tamanho do corpo, limite de chamadas por IP nas rotas sensíveis (login, IA, entrevista, convite) e limites de uso por plano.

**Chamadas de IA:** timeout, nova tentativa só em falhas temporárias e reserva local quando um provedor não está disponível.

## 5. Modelo de dados

| Entidade | Papel |
|---|---|
| Usuário | conta com papel (candidato, empresa, recrutador, admin) |
| Candidato | perfil, competências, currículo |
| Empresa | tenant; plano (Starter, Pro, Scale, Enterprise) e selo de verificada |
| Vaga | status (rascunho, ativa, pausada, encerrada), salário visível, link curto e embedding |
| Candidatura | etapas: enviada → triagem → visualizada → selecionada → entrevista → proposta → contratado / recusado; nota de aderência; convite de entrevista |
| Sessão de entrevista | conversa com a Aria, vídeo, embedding das respostas; ligada ao candidato quando vem de uma candidatura |
| Mensagem | texto, áudio transcrito ou vídeo |
| DISC, antecedentes, score 360° | resultados ligados à sessão |
| Ficha do gestor | nota de 1 a 5 por requisito e recomendação (avançar, talvez, não avançar), uma por gestor |
| Admissão | checklist da CLT (só o status de cada documento), data de início, conclusão |

Toda migration nova só acrescenta tabelas ou colunas, o que permite voltar à versão anterior do código sem mexer no banco.

## 6. API

REST em `/api/v1`, com erros em português no formato `{ error, code }`.

| Área | Para quê |
|---|---|
| Autenticação | cadastro, login, login Google, sessão atual |
| Vagas | feed, vaga pública, QR Code, publicação e edição, dossiês da vaga |
| Entrevista | iniciar, mensagem de texto ou áudio, vídeo pitch, score, apagar |
| Candidaturas | candidatar-se, pipeline, reanálise, entrevista pela candidatura, convite por código |
| Candidato e empresa | perfil, plano e uso, exclusão de conta |
| Currículo | envio com leitura por IA e download protegido |
| RH | ficha do gestor, banco de talentos, painel, admissões |
| IA e ML | busca semântica, análise de currículo, ranking, mensagens de WhatsApp, modelos de aderência e salário |
| Super admin | estatísticas, empresas, planos, faturamento |

## 7. Fluxos principais

**7.1 Link ou QR da vaga (primeiro contato)**
1. O QR leva para a página da vaga.
2. O candidato informa nome e WhatsApp, e a Aria abre a conversa.
3. As respostas por texto ou áudio vão para o agente, até a entrevista ser concluída.
4. Ao concluir, o score 360° é calculado.

**7.2 Candidatura pelo site**
1. O candidato logado se candidata e recebe a análise de aderência.
2. Quando a empresa move a candidatura para **"Entrevista"**, aparece o botão **"Fazer entrevista com a Aria"** em Minhas candidaturas.
3. A sessão fica ligada à conta dele e pode ser retomada em outro aparelho.

**7.3 Convite por WhatsApp**
1. Na etapa "Entrevista", a empresa gera um convite.
2. A mensagem do WhatsApp sai com um link que abre a entrevista direto e com um código para digitar em `/entrevista`.
3. O link e o código são escritos pelo sistema, não pela IA.
4. Link e código abrem a mesma entrevista do fluxo 7.2.

**7.4 Dossiê e score 360°:** a nota é a média ponderada de quatro partes. A parte de compliance fica fora da conta enquanto a verificação for demonstrativa, e os pesos das outras partes são redistribuídos.

| Parte | Peso |
|---|---|
| Técnico | 40% |
| Comportamental | 25% |
| Compliance | 20% |
| Apresentação | 15% |

**7.5 Ficha do gestor:** a decisão humana registrada, com nota por requisito da vaga e recomendação. É a base para medir a concordância entre a IA e o gestor.

**7.6 Contratação e admissão:**
- Ao marcar "Contratado", abre a admissão com 10 documentos.
- A empresa marca cada documento como recebido ou dispensado e define a data de início.
- A admissão só pode ser concluída com todos os documentos resolvidos.

**7.7 Painel de RH:**
- funil por canal (plataforma × link/QR);
- tempo médio até a contratação;
- horas economizadas, com a premissa declarada de 30 min por entrevista concluída.

**7.8 Planos:**

| Plano | Vagas ativas | Entrevistas por mês |
|---|---|---|
| Starter | 1 | 20 |
| Pro | 5 | 300 |
| Scale | ilimitadas | 1.000 |
| Enterprise | sob contrato | sob contrato |

O bloqueio de vagas acontece ao publicar uma vaga. A entrevista só conta na cota na primeira resposta do candidato: abrir o link não consome cota, e entrevistas em andamento seguem até o fim. A cobrança é manual, pelo super admin.

**7.9 Banco de talentos:** busca por significado (pgvector) só entre pessoas que passaram pelas vagas da própria empresa.

## 8. Inteligência artificial

| Componente | Detalhe |
|---|---|
| Aria | entrevista guiada pela descrição da vaga; responde dúvidas só com os dados oficiais da vaga; encerra depois de cerca de 4 turnos bem-sucedidos |
| Aderência | nota, competências compatíveis e faltantes, sugestões; perfil incompleto não recebe veredito |
| Áudio | transcrição com Whisper large-v3 |
| Busca semântica | embeddings de 768 dimensões em vagas, entrevistas e candidaturas |
| Modelos supervisionados | classificador de aderência e estimador de salário (scikit-learn), hoje treinados com dados sintéticos |
| DISC | autodescrição, não teste psicológico |
| Antecedentes | modo de demonstração, sem consulta a bases oficiais |

**Princípios:** a IA ordena e resume, a pessoa decide; a nota não usa nome, foto, idade nem gênero.

## 9. Frontend

| Página | Quem |
|---|---|
| Início, login e cadastro, privacidade, termos, contato | todos |
| Vagas, Minhas candidaturas, Perfil | candidato |
| Entrevista com a Aria (`/apply/...`) e entrada por convite (`/entrevista`) | candidato |
| Painel de vagas, publicar vaga, pipeline e dossiê, banco de talentos, RH | empresa |
| Super admin | admin |

- Identidade visual única em todo o app.
- Pensado primeiro para o celular.
- Zoom e seleção de texto desativados por decisão de produto, com os campos de formulário liberados.

## 10. Segurança e LGPD

- **Isolamento:** isolamento entre empresas comprovado por teste automático a cada mudança.
- **Segredos:** guardados num cofre da nuvem; uma verificação automática barra segredo no código.
- **Dados guardados só como hash:** senhas, tokens de entrevista e códigos de convite.
- **Arquivos:** vídeo e currículo em armazenamento privado, sem link público.
- **Direitos do titular:** o candidato exclui a conta (o que apaga também as entrevistas dele), apaga uma entrevista ou cancela a candidatura (o que apaga a entrevista daquela vaga) a qualquer momento.
- **Papéis:** a empresa é a controladora e o JobSpark é o operador.
- **Decisão automatizada:** revisão humana registrada (art. 20 da LGPD).
- **Minimização:** a admissão guarda só o status dos documentos, e o CPF é sempre mascarado.
- **Transferência internacional:** informada na Política de Privacidade.

## 11. Qualidade e testes

| Portão | O que garante |
|---|---|
| 82 testes automatizados | segurança, sessão, IA, planos, RH, arquivos, robustez, entrevista e convite; nunca usam o banco real |
| Checagem de tipos (TypeScript) | backend e frontend sem erro de tipo |
| Auditoria a cada commit | barra rota sem autenticação, rota sem checagem de dono, empresa vinda do corpo, segredo e migration destrutiva |
| Testes a cada push | auditoria, testes e tipos |
| Caça-falhas de isolamento | 2 empresas e 2 candidatos novos num banco de teste; 71 checagens |
| Ensaio da API | roteiro contra a versão nova antes de liberar o tráfego e contra a produção depois |

## 12. Infraestrutura e deploy

- **Frontend, API e serviço de ML:** rodam no Google Cloud Run. A API e o ML ficam em São Paulo.
- **Banco:** PostgreSQL gerenciado (Neon), com uma cópia separada para ensaios.
- **Arquivos:** bucket privado em São Paulo.
- **Imagens:** uma imagem por commit, gerada no Cloud Build.

**Como uma mudança chega à produção:**
1. Testes e auditoria.
2. Migrations e caça-falhas no banco de teste.
3. Migrations em produção, conferindo que o banco ficou igual ao código.
4. Versão nova no ar **sem tráfego**.
5. Ensaio nessa versão.
6. 100% do tráfego.
7. Ensaio em produção e registro do deploy com o plano de volta.

Voltar à versão anterior leva segundos: basta trocar o tráfego.

## 13. Limitações conhecidas e próximos passos

| Limitação | Próximo passo |
|---|---|
| Modelos de ML com dados sintéticos | retreinar com dados reais dos pilotos e medir a concordância IA × gestor (Sprint 3) |
| Verificação de antecedentes em demonstração | fornecedor real com consentimento, ou remover |
| Uma pessoa por empresa | várias pessoas por empresa, com papéis |
| Cobrança manual | gateway de pagamento quando houver volume |
| Sem aviso de cadastro novo | notificação ao super admin |
