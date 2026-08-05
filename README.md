# DNS Resolver Blackhole Test

Este artefato acompanha o artigo relacionado à avaliação de resolvedores DNS públicos quanto à sua capacidade de bloquear domínios maliciosos. O script `test_resolvers.sh` automatiza a coleta de listas de domínios maliciosos, consulta cada domínio nos diferentes resolvedores DNS por meio do comando `dig` e registra se o domínio foi bloqueado ou resolvido normalmente, gerando um relatório em formato CSV com os resultados comparativos.

# Estrutura

Este repositório é composto por um único script autocontido:

- `test_resolvers.sh`: script principal, responsável por baixar as listas de domínios maliciosos, consultar os resolvedores DNS e gerar o relatório de resultados.

Ao final da execução, o script gera dois arquivos na pasta de execução:

- `dnsblock_hosts_<data>.txt`: lista consolidada e deduplicada de domínios testados, com sua respectiva fonte de origem.
- `dnsblock_result_<data>.csv`: resultado dos testes, no formato `domínio;fonte;resolvedor_1;resolvedor_2;...`, em que cada coluna de resolvedor contém o IP retornado, sendo esta célula vazia quando o domínio é bloqueado.

# Selos Considerados

Os autores julgam como considerados no processo de avaliação os seguintes selos:

- Artefatos Disponíveis
- Artefatos Funcionais
- Experimentos Reprodutíveis

# Dependências

A seguir, são listadas as dependências necessárias para a execução do `test_resolvers.sh`.

## Hardware

- Sistema operacional Linux, testado em distribuições baseadas em Ubuntu.
- Memória RAM: mínimo 8 GB para uma VM.
- Conexão com a internet ativa, sem bloqueios de saída para a porta 53, utilizada para resolução DNS, e para as portas 443 e 80, utilizadas para download das listas.

O tempo total de execução é limitado principalmente pela latência de rede até os resolvedores DNS consultados. Como o script realiza uma consulta DNS por domínio para cada um dos cinco resolvedores, o tempo de execução pode ser elevado mesmo em máquinas com recursos de hardware superiores aos mínimos indicados.

## Comandos necessários

- `dig`, do pacote `dnsutils`: consulta DNS a um resolvedor específico. 
- `wget`: download das listas de domínios maliciosos.
- `parallel`, do pacote GNU Parallel: executa as consultas DNS em paralelo.
- `ping`, do pacote `iputils-ping`: mede a latência até cada resolvedor.
- `awk` e `sort`, dos pacotes `gawk` e `coreutils`: limpeza e deduplicação das listas de domínios.

### Script de instalação das dependências para Debian e Ubuntu

```
sudo apt-get update
sudo apt-get install -y dnsutils wget parallel iputils-ping gawk coreutils
```

## Recursos externos utilizados

O script depende dos seguintes recursos de terceiros, todos públicos e gratuitos, não sendo necessário nenhum cadastro ou credencial para acessá-los:

- Lista de domínios maliciosos mantida pelo projeto URLHaus, disponível em `https://malware-filter.gitlab.io/malware-filter/urlhaus-filter-hosts.txt`.
- Lista de domínios maliciosos mantida pelo CERT.pl, disponível em `https://hole.cert.pl/domains/v2/domains.txt`.
- Cinco resolvedores DNS públicos: Cloudflare, nos endereços `1.1.1.1` e `1.1.1.2`; Cleanbrowsing, no endereço `185.228.168.168`; AdGuard, no endereço `94.140.14.15`; e Quad9, no endereço `9.9.9.9`.

# Preocupações com segurança

- Rede: o script realiza um grande volume de consultas DNS em paralelo, por meio do GNU Parallel, contra servidores DNS públicos de terceiros. Recomenda-se a execução em uma máquina virtual ou ambiente isolado, de modo a evitar impacto em redes de produção.
- Conteúdo malicioso: o script consulta domínios presentes em listas de indicadores de malware, porém realiza apenas resolução DNS desses domínios. Em nenhum momento é feita uma requisição HTTP ou HTTPS, tampouco download de conteúdo desses domínios.
- Credenciais: nenhuma credencial, chave SSH ou informação sensível é necessária para a execução deste artefato.

# Instalação

Para configurar o ambiente para executar o `test_resolvers.sh`, certifique-se de que as dependências listadas na seção Dependências estejam instaladas.

Clone este repositório na máquina que irá executar o script:

```
git clone https://github.com/jaquelinegon/DNS-Resolver-Analysis.git
```

Acesse o diretório do projeto:

```
cd DNS-Resolver-Analysis
```

Conceda permissão de execução ao script:

```
chmod +x test_resolvers.sh
```

# Teste mínimo

Uma vez que todas as dependências estejam instaladas, execute o script:

```
./test_resolvers.sh
```

O próprio script verifica, logo no início, se todas as dependências estão presentes, interrompendo a execução com uma mensagem de erro caso alguma esteja ausente.

Nos primeiros segundos de execução, observe as seções `### Checking average ping to nameservers.` e `### Testing safe hosts.` do log, nas quais o script mede a latência dos cinco resolvedores e testa a resolução de cinco domínios legítimos e conhecidos.

O resultado esperado é que esses domínios sejam resolvidos normalmente, com IP válido, em todos os resolvedores, e que as linhas correspondentes apareçam no arquivo `dnsblock_result_<data>.csv`. Caso essa etapa seja concluída sem erros, o ambiente está corretamente configurado.

# Experimentos

Este trabalho avalia dois aspectos dos resolvedores DNS testados, conforme apresentado no artigo: a taxa de bloqueio de domínios maliciosos e a latência de resposta, comparando cinco resolvedores DNS públicos frente aos domínios maliciosos coletados das listas do CERT.pl e do URLHaus.

O script inicia verificando as dependências necessárias, baixa e consolida as listas de domínios maliciosos, mede a latência até cada resolvedor, testa a resolução de domínios legítimos conhecidos para validar que os resolvedores estão respondendo corretamente e, por fim, executa em paralelo a consulta de cada domínio malicioso contra os cinco resolvedores, registrando se cada um bloqueou ou resolveu normalmente o domínio.

Conforme apresentado no artigo, os resolvedores testados, as listas de domínios utilizadas e os parâmetros de execução são constantes e estão configurados diretamente no início do script `test_resolvers.sh`, nas variáveis `ns_sp_array`, `ns_ip_array` e nas URLs de download das listas.

O script, por padrão, baixa e testa as listas mais recentes de domínios maliciosos disponíveis no momento da execução. Caso seja necessário reproduzir o teste com uma lista de domínios fixa, deve-se substituir as URLs de download no início do script pelas URLs dos arquivos desejados antes da execução.

Considerando que o objetivo do experimento é analisar a capacidade de bloqueio de cada resolvedor, os resultados de latência e o total de domínios testados variam conforme a data de execução, uma vez que as listas são atualizadas diariamente, e conforme a rede utilizada pela máquina que executa o teste. As especificações de hardware e software da máquina utilizada no artigo estão apresentadas no mesmo.

As seções a seguir apresentam o comando para executar o experimento completo, que reproduz as duas reivindicações apresentadas no artigo. Ambas as reivindicações são verificadas a partir da mesma execução do script, sendo cada uma extraída de uma parte distinta do arquivo de resultados.

```
./test_resolvers.sh
```

Este script realiza o download das listas de domínios maliciosos, mede a latência dos cinco resolvedores e, em seguida, testa cada domínio contra os cinco resolvedores, gerando o arquivo `dnsblock_result_<data>.csv`.

## Reivindicação "Taxa de bloqueio de domínios maliciosos por resolvedor"

Verifica-se que os resolvedores com filtro de segurança bloqueiam uma parcela significativa dos domínios maliciosos testados, enquanto o resolvedor sem filtro resolve normalmente a grande maioria deles, conforme apresentado no artigo.

O script deve ser capaz de classificar, para cada domínio e cada resolvedor, se houve bloqueio ou resolução normal, permitindo o cálculo da taxa de bloqueio por resolvedor.

A partir do arquivo `dnsblock_result_<data>.csv`, cada linha corresponde a um domínio testado, e cada coluna de resolvedor contém o IP retornado, sendo esta célula vazia quando o domínio é bloqueado. É possível calcular, por coluna de resolvedor, o percentual de domínios bloqueados em relação ao total testado, reproduzindo a taxa de bloqueio apresentada no artigo.

## Reivindicação "Comparação de latência entre os resolvedores"

Verifica-se que os resolvedores testados apresentam tempos médios de resposta distintos entre si, conforme apresentado no artigo.

Esta etapa é executada no início da execução do script, na seção `### Checking average ping to nameservers.` do log.

A linha `PING (ms)` do arquivo `dnsblock_result_<data>.csv` apresenta o tempo médio de resposta, em milissegundos, de cada resolvedor, permitindo reproduzir a comparação de latência apresentada no artigo.

## Considerações Finais

Os resultados de taxa de bloqueio e latência variam conforme a data de execução, uma vez que as listas de domínios maliciosos são atualizadas diariamente, e conforme as características de rede da máquina utilizada. Dessa forma, é possível analisar a efetividade de cada resolvedor na filtragem de domínios maliciosos, foco deste trabalho.

# LICENSE

Este projeto está licenciado sob a licença MIT. Consulte o arquivo LICENSE, disponível em `https://github.com/jaquelinegon/DNS-Resolver-Analysis/blob/main/LICENSE`, para mais detalhes.
