# Hive – Formatos de armazenamento
## Existem 2 dimensões principais para definir como será realizado o armazenamento de tabelas no Hive:
### Formato da linha (Row Format) e Formato do arquivo (File Format)

#### Row Format: 
- Define como as linhas e os campos das linhas serão armazenados.
- Essa definição do formato da linha é feita no Hive pelo SerDe (Serializer/Deserializer).
- Quando for criada uma tabela no Hive sem definir o ROW FORMAT, o padrão será um registro (row) por linha.
- Um SerDe encapsula a lógica para converter os bytes não estruturados em um registro.
- SerDes são implementados usando Java. Internamente, o mecanismo Hive usa o InputFormat definido para ler um registro de dados.
- O Hive oferece vários SerDes internos, porém também permite utilizar SerDes de terceiros.
- A biblioteca SerDe fica em org.apache.hadoop.hive.serde2.
- Exemplos de SerDes:
    LazySimpleSerDe: org.apache.hadoop.hive.serde2.lazy SerDe default, formato texto delimitado.
    ColumnarSerDe: org.apache.hadoop.hive.serde2.columnar, LazySimpleSerDe para armazenamento em colunas.
    RegexSerDe: org.apache.hadoop.hive.contrib.serde2 Um SerDe na qual colunas são especificadas por uma expressão regular.
    
#### File Format:
- Um formato de arquivo define como as informações são armazenadas ou codificadas em um arquivo de computador.
- No Hive, refere-se a como os registros são armazenados dentro do arquivo. Como os registros são codificados em um arquivo define um formato de arquivo.
- Esses formatos de arquivo variam principalmente em relação à codificação de dados, taxa de compactação, espaço e I/O de disco.
- Hive oferece suporte aos seguintes formatos de arquivo: TextFile, SequenceFile, RCFile, AvroFile, ORCFile, Parquet e Custom IF/OF.

- Hive – TextFile: Arquivo Texto (TextFile) é o formato default de armazenamento do Hive. Se definirmos uma tabela como TEXTFILE, ela poderá carregar dados com delimitadores (ex.: CSV). Por padrão, se usarmos o formato TEXTFILE, cada linha será considerada como um registro. Útil para visualizar os dados manualmente. Não possui a mesma eficiência de espaço, comparado ao arquivo binário.
Formato: Create table textfile_table (column_specs) stored as textfile;

- Hive – SequenceFile: Arquivo sequencial (Sequence File) - armazena os registros em sequência e é orientado a linha. “O Hadoop é bom para poucos arquivos grandes, e ruim para muitos arquivos pequenos (menor que o tamanho do bloco)”. Muito utilizado para mesclar dois ou mais arquivos pequenos em um arquivo maior. Suporta arquivos com compressão (por registro ou por bloco). Em geral, é processado mais rapidamente que o formato TextFile.
Formato: Create table sequencefile_table (column_specs) stored as sequencefile;

- Hive – RCFile: Arquivo colunar (Record Columnar File – RCFile) é um formato muito similar ao Sequence File, porém os dados são armazenados orientado a colunas. Esse formato primeiro particiona linhas horizontalmente em divisões (splits) de linha e, em seguida, particiona verticalmente cada divisão de maneira colunar. O RCFile combina as vantagens de armazenamento de linha e de coluna para satisfazer a necessidade de rápido carregamento de dados, rápido processamento de query e uso eficiente de espaço de armazenamento.
Formato: Create table RCfile_table (column_specs) stored as rcfile;

- Hive – ORCFile: Arquivo (Optimized Row Columnar - ORC) é um formato que permite armazenar dados de uma maneira mais otimizada do que os outros formatos de arquivo anteriores. O formato ORC reduz o tamanho dos dados originais em até 75% (por exemplo: o arquivo de 100 GB se tornará 25 GB). O uso de arquivos ORC melhora o desempenho quando o Hive está lendo, gravando e processando dados.
Vantagens do ORCFile em relação ao RCFile:
• Gera um único arquivo como saída de cada tarefa, reduzindo a carga do NameNode
• Suporte a tipo de dados, incluindo datetime, decimal e os tipos complexos (struct, list, map e union)
• Compactação block-mode com base no tipo dos dados
• Permite leituras simultâneas do mesmo arquivo usando RecordReaders separados
• Limita a quantidade de memória necessária para ler ou escrever
• Metadados armazenados usando Protocol Buffers, permitindo a adição e remoção de campos
Formato: Create table orc_table (column_specs) stored as orc;

- Hive – Parquet: Desenvolvido na Cloudera em parceria com o Twitter. Baseado na solução de formato de arquivo Dremel ColumnIO utilizada pelo Google. Schema similar ao Protocol Buffers, porém com simplificações (ex.: sem Maps e Lists). A Raiz do schema é um grupo de campos chamado message. Compatível com muitas engines de execução: Apache Hive, Apache Drill, Cloudera Impala, Apache Crunch, Apache Pig, Cascading, Apache e Spark.
Incorpora vários recursos que o tornam adequado para operações no estilo DW:
• Armazenamento colunar. Uma consulta pode examinar e executar cálculos em todos os valores de uma coluna enquanto lê apenas uma pequena
fração dos dados de um arquivo ou tabela de dados.
• Opções flexíveis de compactação. Os dados podem ser compactados com qualquer um dos vários codecs.
• Tamanho de arquivo grande. O layout dos arquivos de dados do Parquet é otimizado para consultas que processam grandes volumes de dados, com arquivos individuais no intervalo de vários megabytes ou até gigabytes.
Formato: Create table parquet_table (column_specs) stored as parquet;

- Hive – Avro File: Avro é um projeto open source que fornece serviços de serialização de dados para o Hadoop. Permite pode trocar dados entre o ecossistema do Hadoop e o programa escrito em qualquer linguagem de programação. Avro mantém metadados a nível de arquivo, não de registro. Armazena os dados em formato binário e permite compactação. Avro utiliza JSON para definir tipos de dados e protocolos e serializa os dados em um formato binário compacto.

- Hive – Custom IF/OF: E se as linhas que queremos no Hive não forem linhas nos arquivos de entrada? Nesse cenário, torna-se necessário ler o arquivo como um todo e decodificá-lo para produzir a saída desejada no Hive. Uma maneira de transformar o arquivo é por meio da criação de classes Hive InputFormat e Record Reader personalizadas. Um InputFormat compatível com o Hive pode ser criado pela criação de classes que implementam e estendem as classes da biblioteca padrão mapreduce:
• CustomTextInputFormat.java - estende o FixedLengthInputFormat. Retorna um CustomTextRecordReader que se conecta ao Hive em tempo de
execução.
• CustomTextRecordReader.java - implementa o RecordReader <LongWritable, BytesWritable>. Lê e descompacta (se necessário) os
arquivos do HDFS. Chama ReadASCIIGridFile para fazer a transformação real.
• ReadASCIIGridFile.java - contém uma classe estática que faz a transformação da entrada (uma matriz de bytes - grade ASCII formatada)
para saída (uma matriz de bytes - formato de linha Hive).

- Hive – Codec: Codec (Compression - Decompression) – Algoritmo para compressão e descompressão dos dados.
• Pontos positivos:
– Permite reduzir espaço necessário para armazenar os dados.
– Melhora desempenho na transferência de dados pela rede.
• Pontos negativos:
– Maior consumo de CPU para leitura e escrita dos dados
• Splittable compression: permite processar o arquivo compactado em paralelo (mais de uma função map) 


# A continuação exemplos de utilização dos diferentes formatos de armazenamento no Hive.

Fontes consultadas: material de apoio do curso Pós Graduação em Análise de Big Data da FIA.
