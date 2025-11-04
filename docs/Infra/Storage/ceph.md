# Ceph: A Solução de Armazenamento Unificada, Escalável e Resiliente
Ceph é uma plataforma de armazenamento de dados definida por software, de código aberto, que se destaca por sua capacidade de oferecer, em uma única solução, armazenamento de objetos, blocos e arquivos. Sua arquitetura distribuída e altamente escalável a torna ideal para ambientes de nuvem, virtualização e grandes volumes de dados, operando sobre hardware de baixo custo (commodity).

A base do Ceph é o RADOS (Reliable Autonomic Distributed Object Store), uma camada de armazenamento de objetos inteligente e autogerenciável. Todos os dados, independentemente do tipo de acesso (bloco, arquivo ou objeto), são armazenados como objetos dentro de um cluster RADOS. Esta abordagem garante a resiliência e a distribuição dos dados de forma eficiente.

## Principais Funcionalidades e Características:

### Armazenamento Unificado: Suporta nativamente três tipos de armazenamento:
* Armazenamento de Objetos (RGW - RADOS Gateway): Oferece uma interface compatível com as APIs S3 e Swift, ideal para armazenamento de dados não estruturados, como backups, arquivos de mídia e dados para aplicações web.
* Armazenamento de Blocos (RBD - RADOS Block Device): Fornece dispositivos de bloco virtuais, semelhantes a discos rígidos de rede, que podem ser utilizados por máquinas virtuais e containers. Destaca-se por funcionalidades como provisionamento dinâmico (thin provisioning), snapshots e clones.
* Armazenamento de Arquivos (CephFS - Ceph File System): Apresenta um sistema de arquivos distribuído compatível com o padrão POSIX, permitindo o acesso compartilhado a arquivos por múltiplos clientes, de forma semelhante a um NAS tradicional, porém com a escalabilidade do Ceph.

### Alta Escalabilidade: 
* O Ceph foi projetado para escalar horizontalmente, permitindo a adição de novos nós de armazenamento (servidores com discos) ao cluster de forma transparente e sem interrupção do serviço. A capacidade e o desempenho podem ser expandidos de petabytes a exabytes.

### Resiliência e Alta Disponibilidade: 
* O sistema é inerentemente tolerante a falhas. O Ceph automaticamente replica os dados em diferentes nós do cluster ou utiliza "erasure coding" para garantir a integridade e a disponibilidade dos dados mesmo em caso de falha de discos ou servidores inteiros. O cluster é autogerenciável e auto-reparável, redistribuindo os dados quando um componente falha.

### Arquitetura Distribuída e sem Ponto Único de Falha:
* Todos os componentes do Ceph, como os Monitores (que mantêm o mapa do cluster) e os OSDs (Object Storage Daemons) (que gerenciam os discos de armazenamento), operam de forma distribuída, eliminando a existência de um ponto único que possa comprometer todo o sistema.

### Custo-Benefício:
* Por ser uma solução de software que opera em hardware comum, o Ceph oferece uma alternativa de baixo custo em comparação com soluções de armazenamento proprietárias e de hardware dedicado.

## Possíveis desvantagens:

### Complexidade da Arquitetura e Curva de Aprendizagem:
* Conhecimento Profundo Necessário: Ceph não é uma solução "plug-and-play". Para administrar um cluster de forma eficaz, é preciso compreender profundamente seus componentes (MONs, OSDs, MGRs, RGW, MDS), a forma como os dados são distribuídos pelo algoritmo CRUSH e os mecanismos de recuperação. A curva de aprendizagem para novos administradores pode ser bastante íngreme.
* Tomada de Decisão Crítica: A configuração inicial e as decisões de arquitetura (ex: tamanho dos placement groups, tipo de replicação vs. erasure coding, escolha do hardware) têm um impacto duradouro no desempenho e na estabilidade do cluster. Corrigir más decisões no futuro pode ser uma tarefa complexa e demorada.

### Problemas de Desempenho (Performance Tuning)
* Identificação de Gargalos: Identificar a causa raiz de um problema de desempenho pode ser muito difícil. O gargalo pode estar na rede, em discos lentos, na configuração do Ceph, na CPU do nó ou até mesmo na aplicação cliente. Ferramentas de monitorização são essenciais, mas a interpretação dos dados exige experiência.
* Fine tuning: O Ceph possui centenas de parâmetros de configuração. Ajustar o desempenho para uma carga de trabalho específica (ex: alta IOPS para bancos de dados vs. alta vazão para backups) é um processo iterativo e complexo que exige testes rigorosos. Um ajuste incorreto pode piorar o desempenho.

### Problemas de Rede
* A Rede é Fundamental: Ceph é extremamente sensível à latência e à largura de banda da rede. Uma rede mal configurada, congestionada ou instável é uma das principais causas de problemas, como OSDs sendo marcados como down incorretamente (flapping OSDs), o que desencadeia processos de recuperação desnecessários e consome recursos.
* Configuração de Rede Dupla: A recomendação de usar uma rede pública (para acesso do cliente) e uma rede de cluster (para tráfego interno de replicação e heartbeat) adiciona uma camada de complexidade na configuração e na gestão da infraestrutura de rede.

### Gestão de "Placement Groups" (PGs)
* Dimensionamento Correto: O número de PGs é um dos parâmetros mais críticos e difíceis de alterar depois que o cluster está em produção com dados. Um número muito baixo pode levar a um desequilíbrio na distribuição de dados, enquanto um número muito alto consome mais recursos de memória e CPU dos OSDs e MONs.
* Estados de Erro: PGs podem entrar em estados de erro (stale, inactive, incomplete) devido a falhas de OSDs ou problemas de rede.

Uma comparação entre Ceph e DRBD encontra-se [aqui](https://docs.google.com/document/d/1Tjxb1lMsqWZ9PxRkBNe0isjEefXy2HFcpCGSYYbTvos/edit?usp=sharing) (não foi completamente revisada por enquanto)
