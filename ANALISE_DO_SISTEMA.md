# Análise do Sistema Master-Zap

## 1. Visão Geral do Sistema
**MasterZap** é uma aplicação web interativa projetada para simular a interface do WhatsApp Web (design "mobile-first" que se expande para desktop). Seu objetivo principal é organizar e apresentar dados públicos e jornalísticos vazados relacionados a conversas de figuras públicas (especificamente Daniel Vorcaro e contatos associados). A proposta é funcionar como um "arquivo interativo" de transparência, similar ao projeto jmailarchive.

O projeto não requer autenticação e não lida com dados privados de usuários; ele serve como um visualizador de dados estruturados e hardcoded.

## 2. Arquitetura (Frontend e Backend)
A aplicação segue uma arquitetura cliente-servidor padrão, mas no momento é altamente simplificada devido ao uso de dados em memória, sem um banco de dados real ativo.

### Frontend
- O frontend é uma SPA (Single Page Application) baseada em **React**.
- Utiliza **Wouter** para roteamento mínimo (apenas a rota principal `/` e fallback para `/not-found`).
- Segue a abordagem de componentes modulares, isolando a barra lateral (Sidebar), a área de chat (ChatArea) e modais específicos (GraphModal e CitationsModal).
- Gerenciamento de estado de requisições feito via **TanStack React Query**, o que facilita o consumo da API do backend.

### Backend
- Construído com **Express.js**.
- Define rotas de API em `/api` para expor os dados.
- O armazenamento atual (camada `storage.ts`) utiliza a implementação **MemStorage**, que mantém os dados de contatos, mensagens e citações "hardcoded" diretamente na memória do servidor.
- O servidor e o cliente (no modo de desenvolvimento via Vite) são servidos na mesma porta, de modo integrado.

## 3. Tecnologias Utilizadas
- **Frontend**: React 18, TypeScript, Tailwind CSS (para estilização rápida e responsiva), Radix UI (para componentes acessíveis não estilizados), Lucide React (ícones), vis-network (para renderização de grafos de conexão), Wouter, React Query.
- **Backend**: Node.js, Express.js.
- **Ferramentas de Build/Tooling**: Vite, esbuild, TypeScript (tsc / tsx).
- **Esquemas e Validação**: Zod, Drizzle ORM (presentes no esquema `schema.ts`, embora o armazenamento atual seja em memória e não conecte de fato a um banco Postgres na implementação ativa `MemStorage`).

## 4. Modelo de Dados
Os dados estão modelados usando o Drizzle ORM no arquivo `schema.ts`, mesmo sendo consumidos majoritariamente a partir da memória. As principais entidades são:

- **Contacts (Contatos)**: Representa os usuários/grupos com quem há conversas. Contém: ID, Nome, URL do Avatar, Última Mensagem, Horário da Última Mensagem e Status de Online.
- **Messages (Mensagens)**: Representa o histórico de conversas. Relaciona-se com `Contacts` através do `contactId`. Identifica se o remetente é o dono do dispositivo (`sender: "me"`) ou o contato (`sender: "contact"`), texto da mensagem, horário e grupo de data.
- **Citations (Citações)**: Uma tabela que mantém o contexto das menções nas conversas. Possui Nome, Contexto e Fonte.

## 5. Funcionalidades Principais
1. **Chat estilo WhatsApp**: Uma interface que simula perfeitamente o WhatsApp Web, permitindo listar conversas do lado esquerdo e visualizar as mensagens de um contato à direita.
2. **Visualização de Grafo (Modo Grafo)**: Uma janela modal (utilizando `vis-network`) que exibe conexões e relacionamentos entre os diferentes contatos listados.
3. **Modal de Citações**: Apresenta menções ou citações importantes extraídas das conversas.
4. **Busca (Barra Principal)**: Permite pesquisar nomes, mensagens ou palavras-chave (embora a lógica de filtro dependa da implementação do frontend).
5. **Design Responsivo**: No mobile exibe ou a lista de contatos ou a tela de chat; no desktop expande exibindo ambos lado a lado com a barra verde de fundo característica do WhatsApp.

## 6. Pontos Fortes e Oportunidades de Melhoria

### Pontos Fortes
- **UI/UX Altamente Familiar**: O design se baseia no WhatsApp, o que remove qualquer curva de aprendizado para o usuário.
- **Separação Clara de Responsabilidades**: O código está bem organizado entre rotas do servidor, camada de armazenamento (storage), esquemas compartilhados (`shared/schema.ts`) e componentes UI do React.
- **Performance**: Por carregar dados da memória sem latência de banco de dados, a renderização e o tempo de resposta das APIs são quase instantâneos.

### Oportunidades de Melhoria (Débitos Técnicos e Evolução)
- **Persistência de Dados Real**: O projeto atualmente usa `MemStorage` com dados fixos no código. Para escalabilidade e facilidade de atualização, os dados devem ser movidos para um banco de dados real (como PostgreSQL, considerando que o Drizzle já está configurado no repositório).
- **Tratamento de Dados Massivos**: A longo prazo, carregar todo o arquivo de mensagens diretamente da memória pode consumir muita RAM e dificultar a manutenção se os vazamentos possuírem milhares ou milhões de registros.
- **Paginação / Infinite Scroll**: Atualmente as mensagens de um contato parecem ser retornadas de uma vez. Implementar paginação ou "infinite scroll" na visualização do chat seria ideal para grandes históricos de conversa.
- **Falta de Testes Automatizados**: A análise do `package.json` revela a falta de frameworks de testes como Jest, Vitest ou Cypress. Adicionar testes unitários e e2e (end-to-end) tornaria a aplicação mais robusta.
- **Funcionalidade de Busca Otimizada**: Caso o projeto migre para banco de dados, deve ser considerado o uso de full-text search no backend para pesquisas em vez de busca client-side.
