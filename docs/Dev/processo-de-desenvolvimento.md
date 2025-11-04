## 1. O Fluxo de Desenvolvimento

Nosso processo segue um fluxo claro, desde a concepção de uma tarefa até a sua entrega. Cada etapa foi desenhada para garantir que o trabalho seja feito de forma organizada e com pontos de verificação de qualidade.

**Fluxo Simplificado:** 
![Diagrama BPMN com fluxo de desenvolvimento simplificado](./assets/Fluxo%20Simplificado.svg)

### Etapas do Fluxo:

1. **Seleção da Tarefa:** O desenvolvedor pega uma tarefa (seja uma _feature_, _bugfix_ ou _chore_) do nosso github projects. A tarefa deve ter uma descrição clara do que precisa ser feito.
    
2. **Criação da Branch:** A partir da branch principal de desenvolvimento (`develop`), crie uma nova branch para trabalhar.
    
    - **Padrão de Nomenclatura:** Use o formato `tipo/id-da-task-descricao-curta`.
    - **Exemplos:**
        
        - `feat/123-criar-endpoint-de-login`
            
        - `fix/125-corrigir-validacao-de-email`
            
3. **Desenvolvimento e Commits:** Escreva o código necessário para completar a tarefa.
    
    - **Commits Atômicos:** Faça commits pequenos e focados. Cada commit deve representar uma pequena parte lógica do trabalho.
        
    - **Padrão de Mensagem de Commit:** Use o padrão **Conventional Commits**. Isso nos ajuda a automatizar a geração de changelogs e a entender o histórico do projeto. Os tipos (`feat`, `fix`, `docs`, etc.) devem corresponder às flags usadas no título do PR (`[FEAT]`, `[FIX]`, `[DOCS]`).
        
        - `feat:` para novas funcionalidades.
            
        - `fix:` para correções de bugs.
            
        - `docs:` para mudanças na documentação.
            
        - `style:` para formatação de código.
            
        - `refactor:` para refatorações que não alteram a funcionalidade.
            
        - `test:` para adição ou correção de testes.
            
    - **Exemplo de commit:** `feat: adiciona autenticação via JWT no endpoint /auth`
        
4. **Abertura do Pull Request (PR):** Quando o desenvolvimento estiver concluído, abra um PR da sua branch para a branch `develop`.
    
5. **Code Review:** O PR passa pelo nosso processo de revisão (detalhado na Seção 3).
    
6. **Merge:** Após a aprovação, o PR é "squashed" (agrupado em um único commit) e mergeado na branch `develop`.
    
7. **Deploy:** O código na `develop` é automaticamente enviado para o ambiente de desenvolvimento. O deploy para produção é feito a partir da branch `main`, que recebe as atualizações da `develop` em momentos planejados.
    

## 2. Padrões e Boas Práticas

Para manter nosso código limpo e consistente, seguimos algumas regras essenciais.

### Formatação de Código

A consistência no estilo do código é fundamental para a legibilidade.

- **Ferramentas Automáticas:** Usamos formatadores e linters para garantir o padrão sem esforço manual.
    
    - **Exemplos:** Formatadores do .NET (para C#), Prettier e ESLint (para Vue.js com Tailwind CSS).
        
- **Execução Automática:** Essas ferramentas devem ser executadas automaticamente antes de cada commit, usando um **hook de pre-commit** (como Husky ou pre-commit). Isso garante que nenhum código fora do padrão chegue ao repositório.
    

### Comentários e Documentação

O código deve ser o mais claro possível, mas comentários são necessários para explicar o "porquê", não o "o quê".

- **Lógica Complexa:** Se uma lógica de negócio ou algoritmo for complexo, adicione um comentário explicando o motivo daquela abordagem. Antes de comentar, sempre se pergunte: "É possível refatorar este código para torná-lo mais simples e autoexplicativo?".
    
- **Comentários de Documentação:** Toda função ou método público deve ter uma breve documentação explicando o que faz, seus parâmetros e o que retorna.
    

**Exemplo de Comentário de Documentação (C#):**

```cs
/// <summary>
/// Verifica as credenciais de um usuário no banco de dados.
/// </summary>
/// <param name="email">O email fornecido pelo usuário.</param>
/// <param name="senha">A senha não criptografada fornecida pelo usuário.</param>
/// <returns>Retorna true se as credenciais forem válidas, caso contrário, false.</returns>
public bool AutenticarUsuario(string email, string senha)
{
    // ... lógica da função
}
```

## 3. O Processo de Pull Request e Code Review

O Code Review é nossa principal ferramenta para garantir a qualidade do código, compartilhar conhecimento e mentorar uns aos outros.

### Criando um Bom Pull Request

- **Tamanho Limitado:** Mantenha os PRs pequenos e focados em uma única tarefa. **Um PR ideal tem menos de 300 linhas de alteração.** PRs grandes são difíceis e demorados de revisar, o que leva a revisões de baixa qualidade.
    
- **Descrição Clara:** A descrição do PR é a primeira impressão do revisor. Use o template padrão do nosso repositório para garantir que todas as informações necessárias estejam presentes.
    

**Template de Pull Request:**

```md
> 🚨 **NÃO ESQUEÇA DE ADICIONAR A [FLAG] NO TÍTULO DO SEU PR!**

## Tipo de Mudança
Marque com um "x" o tipo de mudança que este PR introduz:
- [ ] [FEAT]: Nova funcionalidade
- [ ] [FIX]: Correção de bug
- [ ] [HOTFIX]: Correção de falha crítica em produção
- [ ] [REFACTOR]: Refatoração de código
- [ ] [DOCS]: Atualização de documentação
- [ ] Outro (descreva abaixo)

## Descrição
Descreva de forma clara e objetiva as alterações propostas neste PR, incluindo o motivo e o contexto.

## Issues relacionadas
- Resolve leds-conectafapes/devops-project#[NUMERO-ISSUE]
```

### O Processo de Revisão em Duas Etapas

Adotamos um modelo que acelera o feedback e treina todo o time.

**Fluxo de Code Review:** 
![Diagrama BPMN com fluxo de code review](./assets/Fluxo%20Code%20Review.svg)


#### Etapa 1: Revisão por Pares (Peer Review)

- **Quem:** Todo PR deve ser revisado por **pelo menos um outro desenvolvedor júnior**.
    
- **O quê:** O revisor deve focar em entender o código e garantir sua qualidade. Use a checklist abaixo como guia.
    
- **Como:** Os comentários devem ser construtivos. Em vez de dizer "Isso está errado", sugira uma alternativa: "O que você acha de usarmos a função X aqui? Ela parece simplificar o código.".
    

**Checklist do Revisor:**

- [ ] O código resolve o problema descrito na tarefa?
    
- [ ] A lógica está clara e fácil de entender?
    
- [ ] Existem bugs óbvios ou casos de borda não tratados?
    
- [ ] O código segue os padrões de formatação e estilo da equipe?
    
- [ ] Foram adicionados testes para a nova funcionalidade/correção?
    
- [ ] A documentação (comentários, docstrings) está clara e suficiente?
    
- [ ] O PR é pequeno e focado?
    

#### Etapa 2: Revisão do Líder (Review da Review)

- **Quem:** O líder técnico.
    
- **O quê:** O foco do líder não é apenas o código, mas **a qualidade da revisão feita pelos pares**.
    
- **Como:** O líder avalia:
    
    - A revisão por pares foi completa? O revisor júnior deixou passar algo importante?
        
    - O feedback foi construtivo e claro?
        
    - Além disso, o líder adiciona comentários sobre aspectos de mais alto nível, como arquitetura, design patterns e impacto a longo prazo da solução.
        

Este modelo tem um duplo benefício: os desenvolvedores júniors praticam a habilidade de dar e receber feedback, e o líder pode focar seu tempo em pontos estratégicos, escalando a cultura de qualidade no time.