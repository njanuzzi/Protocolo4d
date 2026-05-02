# Protocolo 4D — Product Requirements Document
**MVP v1.0 | Maio 2026**

---

## 1. Visão Geral

O **Protocolo 4D** é um sistema web privado que conecta terapeutas e clientes por meio de diários clínicos estruturados. O cliente registra check-ins diários respondendo perguntas pré-definidas; o terapeuta acompanha os padrões, gera relatórios semanais com apoio de IA e os disponibiliza ao cliente.

> Um portal privado onde clientes registram check-ins diários e terapeutas acompanham padrões semanais por meio de relatórios publicados.

| Campo | Valor |
|---|---|
| Nome do produto | Protocolo 4D |
| Versão | MVP 1.0 |
| Público-alvo | Terapeuta (admin) + Clientes cadastrados |
| Stack recomendada | Bolt.new + Supabase + Vercel |
| Autenticação | Terapeuta: e-mail + senha / Cliente: link mágico |
| Idioma da interface | Português (Brasil) |

---

## 2. Perfis de Usuário

### 2.1 Terapeuta (Admin)
- Único usuário com acesso total ao sistema
- Cadastra e gerencia clientes
- Cria e ativa diários
- Lê respostas e histórico de todos os clientes
- Gera, edita e publica relatórios semanais
- Login: e-mail + senha

### 2.2 Cliente
- Acessa apenas o próprio diário e relatórios publicados
- Responde o diário ativo uma vez por dia
- Não vê dados de outros clientes
- Login: link mágico enviado por e-mail (sem senha)

---

## 3. Funcionalidades do MVP

### 3.1 Autenticação

| Funcionalidade | Comportamento esperado | Quem usa |
|---|---|---|
| Login terapeuta | E-mail + senha. Redireciona para dashboard admin. | Terapeuta |
| Login cliente | Terapeuta insere o e-mail do cliente. Cliente recebe link mágico. Clica e entra direto. | Cliente |
| Logout | Encerra sessão e redireciona para login. | Ambos |
| Proteção de rotas | Cada perfil só vê suas telas. Acesso direto por URL é bloqueado. | Sistema |

---

### 3.2 Gerenciamento de Clientes

| Ação | Campos | Observação |
|---|---|---|
| Cadastrar cliente | Nome completo, e-mail, data de início | Terapeuta cadastra; cliente recebe convite por e-mail |
| Listar clientes | Nome, status (ativo/inativo), diário ativo | Visualização somente para terapeuta |
| Desativar cliente | Botão de toggle | Cliente perde acesso ao portal |

---

### 3.3 Gerenciamento de Diários

Cada diário é um conjunto de perguntas. Pode existir vários diários cadastrados, mas **apenas um ativo por vez**. O diário ativo é o que o cliente responde.

| Ação | Detalhes |
|---|---|
| Criar diário | Nome do diário + lista de perguntas (mínimo 1, máximo 10). Cada pergunta tem: texto + tipo de resposta (texto livre, escala 1–5, sim/não). |
| Ativar diário | Apenas um diário pode estar ativo. Ativar um desativa o anterior automaticamente. |
| Visualizar diário | Terapeuta vê todas as perguntas e o status (ativo / inativo). |
| Editar diário | Somente diários inativos podem ser editados. |

---

### 3.4 Diário Diário (Preenchimento pelo Cliente)

| Regra | Detalhe |
|---|---|
| Frequência | Uma resposta por dia por cliente. |
| Conteúdo exibido | Apenas as perguntas do diário ativo no momento do preenchimento. |
| Bloqueio | Se já respondeu hoje, a tela mostra confirmação e bloqueia novo envio. |
| Histórico | Cliente vê seus próprios registros anteriores (data + respostas). |
| Armazenamento | Cada resposta salva: cliente_id, diario_id, data, pergunta_id, resposta, timestamp. |

Tipos de pergunta suportados:
- **Texto livre:** campo de texto aberto
- **Escala 1–5:** botões de seleção numérica
- **Sim/Não:** dois botões de escolha exclusiva

---

### 3.5 Painel do Terapeuta — Leitura de Respostas

- Rota: `/clients/:id/entries`
- Seleciona cliente → vê histórico de respostas com data
- Filtro por período (semana / mês / personalizado)
- Visualização: tabela com data + pergunta + resposta
- Exportação opcional: download CSV do histórico

---

### 3.6 Geração de Relatório Semanal com IA

**Rota:** `/clients/:id/report/new`

Fluxo em 4 etapas, exibido como barra de progresso na tela:

| Etapa | Ação | Responsável |
|---|---|---|
| 1 — Selecionar período | Terapeuta escolhe cliente + data de início e fim do período | Terapeuta |
| 2 — Gerar com IA | Sistema coleta todas as respostas do período e envia para a API de IA. IA retorna rascunho estruturado. | Sistema + IA |
| 3 — Revisar e editar | Terapeuta lê o rascunho na tela. Pode editar diretamente no campo de texto, baixar em .docx para edição avançada, ou subir um .docx editado. | Terapeuta |
| 4 — Publicar | Terapeuta clica em "Publicar para cliente". Relatório fica visível no portal do cliente. | Terapeuta |

**Layout da tela (duas colunas):**

Coluna esquerda — Configuração:
- Cliente selecionado (somente leitura)
- Período: dois campos de data (início e fim)
- Diário utilizado: nome + quantidade de perguntas
- Respostas no período: ex. "6 de 6 dias respondidos"
- Botão "Regerar rascunho": gera novo rascunho descartando o atual

Coluna direita — Rascunho:
- Badge indicando que o conteúdo foi gerado por IA e requer revisão
- Chips de padrões detectados automaticamente (ex: "Ansiedade elevada qua/qui", "Sono regular")
- Campo de texto editável com o rascunho completo
- Botão "Baixar .docx": exporta o rascunho atual em Word
- Botão "Subir .docx editado": substitui o rascunho pelo arquivo enviado
- Botão "Publicar para cliente": publica o relatório

**Formato do relatório gerado pela IA:**
```
Período: [data início] a [data fim]
Cliente: [nome]

PADRÕES OBSERVADOS
[Síntese geral das respostas da semana]

VARIAÇÕES SIGNIFICATIVAS
[Oscilações em humor, sono, comportamento]

PONTOS DE ATENÇÃO
[Pontos relevantes — sem diagnóstico]

PRÓXIMOS PASSOS SUGERIDOS
[Lista de ações recomendadas]
```

**Regras de negócio:**

| Regra | Detalhe |
|---|---|
| Período mínimo | Pelo menos 1 resposta no período para gerar relatório |
| Edição do rascunho | Campo de texto livre — terapeuta pode reescrever qualquer parte |
| Upload de .docx | Substitui o campo de texto pelo conteúdo do arquivo enviado |
| Publicação | Apenas o terapeuta publica. Cliente não vê rascunhos. |
| Histórico | Relatórios publicados ficam acessíveis em /clients/:id/reports |
| Despublicação | Terapeuta pode despublicar a qualquer momento |

---

### 3.7 Publicação e Acesso do Cliente ao Relatório

| Regra | Detalhe |
|---|---|
| Visibilidade | Cliente só vê relatórios marcados como "publicado". |
| Formato | Leitura direta no portal (HTML renderizado) OU download do .docx. |
| Histórico | Cliente vê todos os relatórios publicados anteriores. |
| Controle | Terapeuta pode despublicar se necessário. |

---

## 4. Banco de Dados (Supabase — PostgreSQL)

### Tabelas

| Tabela | Campos principais | Relação |
|---|---|---|
| users | id, email, role (therapist \| client), name, active, created_at | Base de autenticação Supabase Auth |
| diaries | id, name, is_active, created_at, updated_at | Um diário ativo por vez |
| diary_questions | id, diary_id, order, text, type (text \| scale \| yesno) | N perguntas por diário |
| diary_entries | id, user_id, diary_id, date, created_at | Uma entrada por cliente por dia |
| entry_answers | id, entry_id, question_id, answer_text, answer_value | N respostas por entrada |
| reports | id, user_id (cliente), period_start, period_end, content_text, file_url, published, created_at | Relatório semanal por cliente |

### Regras de Segurança — Row Level Security (RLS)

- **Terapeuta:** lê e escreve tudo
- **Cliente:** lê apenas suas próprias entries e relatórios publicados
- **Cliente:** nunca acessa dados de outros clientes

---

## 5. Mapa de Telas

### 5.1 Telas do Terapeuta

| Tela | Rota | Função principal |
|---|---|---|
| Login | /login | Acesso com e-mail + senha |
| Dashboard | /dashboard | Visão geral: clientes ativos, diário ativo, últimos relatórios |
| Clientes | /clients | Listagem + cadastro + ativação/desativação |
| Diários | /diaries | Listagem de diários + criação + ativação |
| Criar Diário | /diaries/new | Formulário com nome + perguntas |
| Respostas do Cliente | /clients/:id/entries | Histórico de preenchimentos com filtro |
| Gerar Relatório | /clients/:id/report/new | Selecionar período → gerar com IA → editar → publicar |
| Relatórios | /clients/:id/reports | Histórico de relatórios + status publicado/rascunho |

### 5.2 Telas do Cliente

| Tela | Rota | Função principal |
|---|---|---|
| Acesso via link mágico | /auth/magic-link | Login sem senha |
| Diário do Dia | /diary | Perguntas do diário ativo + envio |
| Histórico | /diary/history | Registros anteriores do próprio diário |
| Relatórios | /reports | Relatórios publicados pelo terapeuta |

---

## 6. Matriz de Permissões

| Recurso | Terapeuta | Cliente |
|---|---|---|
| Ver todos os clientes | ✅ | ❌ |
| Cadastrar clientes | ✅ | ❌ |
| Criar/editar diários | ✅ | ❌ |
| Ver respostas de clientes | ✅ | ❌ (só as próprias) |
| Responder diário do dia | ❌ | ✅ |
| Ver histórico próprio | ❌ | ✅ |
| Gerar relatório com IA | ✅ | ❌ |
| Publicar relatório | ✅ | ❌ |
| Ver relatórios publicados | ✅ (todos) | ✅ (só os seus) |
| Download .docx do relatório | ✅ | ✅ (somente publicados) |

---

## 7. Fases de Desenvolvimento

| Fase | Objetivo | Entrega |
|---|---|---|
| Fase 1 — MVP Funcional | Sistema completo: autenticação, diário, histórico, relatório manual | Terapeuta consegue usar com clientes reais |
| Fase 2 — IA no Relatório | Integrar API de IA para gerar rascunho automático do relatório semanal | Fluxo: gerar → editar → publicar |
| Fase 3 — Multi-terapeuta (SaaS) | Cadastro de múltiplos terapeutas, cada um com seus clientes | Produto escalável para outros profissionais |

---

## 8. Stack Técnica

| Componente | Ferramenta | Função |
|---|---|---|
| Builder | Bolt.new | Geração do código com IA |
| Banco de dados + Auth | Supabase | PostgreSQL + autenticação + RLS |
| Hosting | Vercel (Fase 1 final) | Deploy público estável |
| IA para relatório | Claude API ou OpenAI GPT-4 | Geração do rascunho semanal |
| Relatório Word | docx.js ou equivalente | Download do relatório em .docx |

---

## 9. Design Visual

| Elemento | Valor |
|---|---|
| Cor primária | Azul petróleo `#1B4B5A` |
| Cor secundária | Bege `#E8DCC8` |
| Cor de destaque | Dourado `#C9A84C` |
| Tom da interface | Clínico, sóbrio, premium feminino — sem elementos motivacionais |
| Idioma | Português (Brasil) |
| Tipografia sugerida | Display: Playfair Display / Corpo: DM Sans |

---

## 10. Prompt para o Bolt.new

Cole o texto abaixo como primeiro prompt no Bolt.new:

---

```
Crie um sistema web chamado Protocolo 4D com as seguintes especificações:

STACK: React + Supabase (auth, banco, storage) + Tailwind CSS.

PERFIS:
- Terapeuta (admin): login com e-mail + senha
- Cliente: login via link mágico (magic link) enviado por e-mail

BANCO DE DADOS (Supabase):
- users: id, email, role, name, active, created_at
- diaries: id, name, is_active, created_at
- diary_questions: id, diary_id, order, text, type [text | scale | yesno]
- diary_entries: id, user_id, diary_id, date, created_at
- entry_answers: id, entry_id, question_id, answer_text, answer_value
- reports: id, user_id, period_start, period_end, content_text, file_url, published, created_at

REGRAS DE SEGURANÇA (RLS):
- Terapeuta lê e escreve tudo
- Cliente lê apenas seus próprios dados (entries + relatórios publicados)

FUNCIONALIDADES DO TERAPEUTA:
1. Dashboard com visão geral
2. Cadastro e listagem de clientes
3. Criação de diários com perguntas (tipos: texto, escala 1-5, sim/não)
4. Ativação de diário (apenas um ativo por vez)
5. Visualização de respostas por cliente com filtro por período
6. Geração de relatório semanal (rascunho de texto editável gerado por IA)
7. Upload de arquivo .docx como relatório final (opcional)
8. Publicação do relatório para o cliente

FUNCIONALIDADES DO CLIENTE:
1. Login por link mágico
2. Responder o diário ativo (uma vez por dia, bloqueio automático se já respondido)
3. Ver histórico das próprias respostas
4. Ver relatórios publicados pelo terapeuta
5. Download do relatório em .docx

TELA DE GERAÇÃO DE RELATÓRIO (layout duas colunas):
- Coluna esquerda: seleção de cliente, período (data início e fim), resumo das respostas no período, botão "Regerar rascunho"
- Coluna direita: badge "Gerado por IA", chips com padrões detectados, campo de texto editável com o rascunho, botão baixar .docx, botão subir .docx editado, botão "Publicar para cliente"
- Barra de progresso com 4 etapas: Selecionar período → Gerar com IA → Revisar e editar → Publicar

DESIGN:
- Paleta: azul petróleo #1B4B5A + bege #E8DCC8 + dourado #C9A84C
- Tom: clínico, sóbrio, premium — sem elementos motivacionais
- Tipografia: Playfair Display para títulos, DM Sans para corpo
- Idioma: Português (Brasil) em toda a interface

REGRAS IMPORTANTES:
- Cliente nunca vê dados de outros clientes
- Cliente nunca vê relatórios não publicados
- Apenas um diário pode estar ativo por vez
- Cliente só pode responder o diário uma vez por dia
```

---

## 11. Critérios de Aceite do MVP

O MVP está pronto quando todos os itens abaixo funcionarem:

- [ ] Terapeuta consegue criar conta e fazer login
- [ ] Terapeuta consegue cadastrar um cliente
- [ ] Cliente recebe e-mail com link mágico e consegue acessar o portal
- [ ] Terapeuta cria um diário com perguntas e o ativa
- [ ] Cliente responde o diário e não consegue responder duas vezes no mesmo dia
- [ ] Terapeuta vê as respostas do cliente
- [ ] Terapeuta gera um rascunho de relatório semanal
- [ ] Terapeuta edita e publica o relatório
- [ ] Cliente vê o relatório publicado
- [ ] Dados de um cliente não são visíveis para outro

---

*Protocolo 4D · Documento confidencial · v1.0*
