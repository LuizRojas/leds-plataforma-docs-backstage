# Observabilidade em Sistemas .NET com OpenTelemetry

Este guia apresenta como aplicar **observabilidade em aplicações .NET** utilizando **OpenTelemetry**, de forma padronizada entre os sistemas do projeto ConectaFapes.

O objetivo é garantir rastreamento consistente entre serviços distribuídos, facilitando a análise de falhas, gargalos e fluxos de negócio.

---

## Conceitos

- **OpenTelemetry (OTEL)**: padrão aberto para coleta de métricas, logs e traces.  
- **ActivitySource**: ponto de entrada para criar spans customizados no código .NET.  
- **Span**: unidade de rastreamento que representa uma operação.  
- **Trace**: encadeamento de spans que mostram o caminho de uma requisição.

---

## Configuração do Projeto

### Dependências NuGet

No `.csproj`, adicione:

```xml
<ItemGroup>
  <PackageReference Include="OpenTelemetry" Version="1.*" />
  <PackageReference Include="OpenTelemetry.Extensions.Hosting" Version="1.*" />
  <PackageReference Include="OpenTelemetry.Instrumentation.AspNetCore" Version="1.*" />
  <PackageReference Include="OpenTelemetry.Instrumentation.Http" Version="1.*" />
  <PackageReference Include="OpenTelemetry.Exporter.OpenTelemetryProtocol" Version="1.*" />
</ItemGroup>
````

### Registro do OpenTelemetry

No `Program.cs`:

```csharp
using OpenTelemetry.Resources;
using OpenTelemetry.Trace;

builder.Services.AddOpenTelemetry()
    .WithTracing(tracerProviderBuilder => tracerProviderBuilder
        .AddSource("<NOME_DO_SISTEMA>") // Definir nome único por serviço
        .AddAspNetCoreInstrumentation()
        .AddHttpClientInstrumentation()
        .SetResourceBuilder(ResourceBuilder.CreateDefault()
            .AddService("<NOME_DO_SISTEMA>"))
        .AddOtlpExporter());
```

O valor de `<NOME_DO_SISTEMA>` deve ser padronizado com o nome do repositório ou serviço.

---

## Criando um ActivitySource Central

Cada serviço deve ter um `TracingSources.cs`:

```csharp
using System.Diagnostics;

namespace <NamespaceDoProjeto>.Shared.Tracing
{
    public static class TracingSources
    {
        public static readonly ActivitySource Activity =
            new ActivitySource("<NOME_DO_SISTEMA>");
    }
}
```

---

## Instrumentação do Código

Utilizar spans manuais em pontos críticos de negócio.

### Exemplo 1 — Operação de Leitura

```csharp
var leituraActivity =
    TracingSources.Activity.StartActivity("LeituraArquivo", ActivityKind.Internal);

try
{
    // lógica de leitura
    leituraActivity?.SetTag("arquivo.nome", nomeArquivo);
    leituraActivity?.SetTag("arquivo.tamanho", conteudo.Length);
}
finally
{
    leituraActivity?.Dispose();
}
```

### Exemplo 2 — Operação de Negócio

```csharp
var cadastroActivity =
    TracingSources.Activity.StartActivity("CadastroEntidade", ActivityKind.Internal);

cadastroActivity?.SetTag("entidade.id", entidade.Id);
cadastroActivity?.SetTag("cadastro.sucesso", true);

// ...
cadastroActivity?.Dispose();
```

### Exemplo 3 — Tratamento de Erros

```csharp
var erroActivity =
    TracingSources.Activity.StartActivity("OperacaoComErro", ActivityKind.Internal);

erroActivity?.SetTag("erro.tipo", ex.GetType().Name);
erroActivity?.SetTag("erro.mensagem", ex.Message);
erroActivity?.SetStatus(ActivityStatusCode.Error);

erroActivity?.Dispose();
```

---

## Boas Práticas

* Definir **nomes claros de spans** (ex.: `LeituraArquivo`, `CadastroEntidade`).
* Adicionar **tags relevantes** (`SetTag`) para enriquecer os spans.
* Marcar erros com `SetStatus(ActivityStatusCode.Error)`.
* Utilizar `AddEvent(new ActivityEvent("evento"))` para pontos importantes dentro de um span.
* Evitar spans desnecessários; foque em operações críticas.
* Padronizar os nomes de serviço (`service.name`) conforme o repositório.

---

## Checklist para Replicação em Outros Sistemas

1. Adicionar os pacotes NuGet de OpenTelemetry.
2. Criar um `TracingSources` com `ActivitySource`.
3. Configurar `Program.cs` com `AddOpenTelemetry().WithTracing(...)`.
4. Ativar instrumentações automáticas (AspNetCore, HttpClient, EF Core se necessário).
5. Criar spans manuais em pontos críticos de negócio.
6. Adicionar tags e eventos nos spans.
7. Propagar contexto em comunicações entre serviços (gRPC, filas, APIs).
8. Definir `service.name` padronizado por repositório.

---

## Conclusão

Seguindo este padrão, todos os serviços .NET do ConectaFapes terão rastreamento consistente, permitindo correlação entre sistemas distribuídos e maior visibilidade do fluxo de execução.
