#### PROJETO EM EDIÇÃO 
<p></p>

# Combinando Pesquisa Textual e IA
## Busca semântica e textual em documentos
Essa é uma proposta de pesquisa textual implementada com recursos nativos do banco de dados MemSQL com o uso de dicionário de sinônimos, recursos fonéticos e distância entre termos pesquisados. É uma pesquisa que tenta ir além do que pesquisas comuns fazem, pois não tem o objetivo de trazer grandes volumes de resultados, mas resultados precisos. 

#### em breve...
<p>Estarão disponíveis nesse repositório:</p>
<ul>
  <li>Scripts em python necessários para treinamento e vetorização dos documentos, o que permite ir além da busca textual, realizando uma busca semântica nos textos.</li>
  <li>Procedures e funções criadas para o <b>MemSQL</b>: responsáveis pelo pré-processamento dos textos para armazenamento no formato compatível com a pesquisa textual; funções responsáveis pela análise dos critérios de pesquisa; funções para identificação dos pesos dos termos (tfidf) em cada documento; e funções e procedures acessórias.
  <li>Scripts opcionais em python para inclusão de grandes volumes de dados na base</li>
</ul>
<p>São aplicações simples e práticas de algoritmos de Inteligência Artificial/Machine Learning com o uso de softwares <b>open source</b> e o banco de dados <b>MemSQL</b> disponível na versão community e enterprise.
<p>Para integrar a ferramenta de pesquisa textual a aplicações web ou cliente-servidor é necessário apenas o uso de um cliente <b>MySQL</b>. Para a vetorização de textos para busca semântica é necessário rodar os scripts <b>Python</b> com um modelo treinado a fim de vetorizar novos documentos inseridos na base.</p>

### MemSQL
O MemSQL é um SGBD compatível com o padrão ANSI-SQL, distribuído e altamente escalável, prometendo desempenho máximo para cargas de trabalho transacionais e analíticas de bigdata, usando modelos de dados bem conhecidos. Suporta metadados JSON, comparação de vetores multidimensionais e análise de grandes volumes de dados. Está disponível em https://www.memsql.com, pode ser usado na versão Community ou com suporte pago. Para facilitar o uso desse banco de dados, o acesso é feito por meio de conectores compatíveis com MySQL, apesar do banco de dados não ter qualquer relação de desenvolvimento com o MySQL, como explica o fabricante. 
<p>É possível comparar o MemSQL com outros SGDBs no link: https://db-engines.com/en/system/MemSQL

### Pesquisa textual padrão do MemSQL
O MemSQL possui uma pesquisa textual nativa, com operadores e curingas com performance considerável em milhões de registros textuais. Possui também a capacidade de pesquisa com Expressões Regulares (RegEx) com performance também considerável em volumes de milhões de registros. <b>Apesar</b> dessa capacidade de pesquisa, que assemelha-se ao <b>ElasticSearch</b> em alguns casos, ele não permite filtros mais refinados nos seus critérios de pesquisa nativos. 
<p>RLIKE: https://docs.memsql.com/sql-reference/v6.8/rlike_and_regexp/ 
<p>MATCH: https://docs.memsql.com/sql-reference/v6.8/match/ 
</p>

#### Cursos oficiais disponíveis
<ul><li>https://training.memsql.com/</li></ul>

### Pesquisa textual avançada
Implementei aqui um operador de pesquisa por proximidade dos termos e outros operadores para refinamento de pesquisa. Esse tipo de operador torna-se importante para refinar pesquisas em grande volume de dados, onde não é importante trazer muito resultado, mas um resultado o mais próximo possível do que é procurado. Ferramentas comuns de busca como <b>ElasticSearch</b> e o próprio <b>MemSQL</b> não trazem nativamente esse tipo de operador. 
<p>Essa ideia não é nova, conheci ao longo dos últimos 20 anos vários sistemas que faziam algo parecido. Não há pretensão em competir com qualquer um desses produtos, mas ter algo simples e operacional para quem tiver interesse em personalizar uma busca textual da forma que precisar. 
<p> Esse tipo de pesquisa permite o uso de dicionário de sinônimos em qualquer língua, inclusive o uso de recursos fonéticos. O texto de entrada é pré-processado para evitar não encontrar termos com grafia incorreta de acentos ou termos no singular/plural, bem como números com pontuação ou sem. Por padrão o texto é pesquisado no singular, removendo pronomes oblíquos, mas é possível localizar o termo real usando aspas (exceto acentos que sempre são desconsiderados).
<p> A pesquisa também permite localizar termos pelo dicionário de sinônimos. Ao pesquisar a palavra "genitor", o sistema pesquisa também "pai". A tabela de sinônimos é flexível e facilmente atualizável, permitindo incluir termos em outras línguas se desejado. O uso de sinônimos pode ser ativado ou desativado a cada pesquisa. Ao pesquisar termos entre aspas, o sinônimo é desativado para o termo ou conjunto de termos entre aspas enquanto os outros termos podem ser pesquisados com o uso dos sinônimos.

<p> O pré-processamento envolve:
<ul>
  <li> retirada de <b>acentos</b> </li>
  <li> redução a um <b>pseudosingular</b> ou <b>singular estimado</b>: não é um português perfeito, mas uma singularização para a máquina localizar termos com maior flexibilidade</li>
  <li> remoção de <b>pronomes oblíquos</b>: o pré-processamento transforma "contar-lhe-ia" em "contar", mas a pesquisa entre aspas pode localizar o termo real "contar-lhe-ia"</li>
</ul>

#### Conectores ou operadores de pesquisa
Conectores ou operadores de pesquisa são termos especiais utilizados em sistemas de pesquisa para indicar a relação desejada entre os termos pesquisados. Por exemplo, se é desejado encontrar documentos com a palavra <i>casa</i> e a palavra <i>papel</i>, pode-se escrever o critério de pesquisa como <b>casa papel</b> ou pode-se escrever <b>casa E papel</b>. O operador <b>E</b> está subentendido quando nenhum operador é informado. Para usar termos que são iguais aos operadores, é necessário colocar o termo entre aspas. Ex.: <b>amor e ódio</b> deveria ser escrito como <b>amor "e" ódio</b> para garantir que os três termos existem no texto. Ou também <b>"amor e ódio"</b> para que os termos sejam exigidos nessa sequência, um seguido do outro.  
<ul>
  <li> <b>E</b>: conector padrão, exige a existência do termo no documento</li>
  <li> <b>NÃO</b>: nega a existência de um termo no documento </li>
  <li> <b>OU</b> entre termos: indica que um ou outro termo podem ser encontrados para satisfazer a pesquisa ex.: | "fazer" OU "feito" E casa | realiza uma pesquisa que encontre (fazer ou feito literalmente) e também (casa ou termos que no singular sejam escritos como casa)</li>
  <li> <b>OU</b> com parênteses: permite realizar pesquisas mais complexas. No momento nem todas as combinações de parênteses em pesquisas com OU são possíveis devido ao limite de aninhamentos usados no MemSQL. Ex.: | (casa ADJ5 papel) ou (casa ADJ5 moeda) |. Nesse caso a pesquisa pode ser simplificada como | casa ADJ5 papel ou moeda |</li>
  <li> <b>ADJ</b>n: permite localizar termos que estejam até n termos a frente do primeiro termo. Ex.: | casa ADJ3 papel | vai localizar textos que contenham "casa de papel", "casa papel", "casa feita de papel", mas não localizaria "casa feita de muito papel". </li>
  <li> <b>ADJC</b>n: equivalente ao <b>ADJ</b> padrão, mas obriga que os dois termos estejam presentes no mesmo parágrafo. Não necessariamente a mesma sentença, mas o mesmo parágrafo considerando a quebra <b>/n</b> no texto </li>
  <li> <b>PROX</b>n: semelhante ao <b>ADJ</b>, mas localiza termos posteriores ou anteriores ao primeiro termo pesquisado. Ex.: | casa PROX3 papel | vai localizar textos que contenham "casa de papel", "papel na casa", "papel colado na casa", "casa feita de papel", mas não localizaria "casa feita de muito papel" ou "papel desenhado e colado na casa".</li>
  <li> <b>PROXC</b>n: equivalente ao <b>PROX</b> padrão, mas obriga que os dois termos estejam presentes no mesmo parágrafo. Não necessariamente a mesma sentença, mas o mesmo parágrafo considerando a quebra <b>/n</b> no texto </li>
  <li> <b>COM</b>: obriga que os dois termos pesquisados estejam presentes em um mesmo parágrafo, independente da distância e da ordem. Ex.: | casa COM papel | localiza todos os textos que contenham a palavra casa e a palavra papel, em qualquer ordem e distância, mas que estejam em um mesmo parágrafo. </li>
  <li> <b>MESMO</b>: os documentos podem ser indexados com um tipo único, ou com tipos independentes como, por exemplo: resumo, dados textuais complementares e o texto original. O operador MESMO permite que o documento seja encontrado apenas se os termos pesquisados estiverem em um mesmo tipo do documento. Sem o operador MESMO, o texto será localizado se tiver um termo em um tipo (resumo por exemplo) e outro termo em outro tipo (índice remissivo, por exemplo). O operador MESMO funciona apenas como substituição do operador <b>E</b>, pois os operdores ADJ, ADJC, PROX, PROXC e COM subentendem o uso do operador MESMO por usarem recrusos de distância entre termos. Ex.: | casa MESMO papel | vai localizar textos que contenham "casa" E "papel" no mesmo tipo de documento, caso o termo "casa" esteja no resumo e "papel" esteja no índice, o texto não será localizado.</li>  
</ul>

### Similaridade semântica
Somando-se essa pesquisa textual ao poder de busca vetorial do MemSQL, podemos ter um sistema avançado que consegue localizar em poucos segundos textos semelhantes a um texto paradigma ou textos que contenham determinados critérios refinados. Bem como unir a pesquisa vetorial à pesquisa textual avançada. É uma ferramenta poderosa para busca em documentos acadêmicos, jurídicos etc.
<p>A pesquisa vetorial é nativa do MemSQL, sendo necessário apenas treinar ou usar um modelo já treinado para vetorizar textos (também aplicável a imagens ou outras fontes de dados). Isso pode ser feito de forma relativamente simples usando frameworks como o <b>DOC2VEC</b> do Gensim. A busca vetorial do MemSQL promete, e cumpre, a comparação vetorial em um volume considerável de dados (dezenas de milhões de vetores) em segundos ou até frações de segundos.
<p> Em breve será disponibilizado o código com o passo-a-passo de como treinar um modelo para vetorizar textos para pesquisa semântica.
<p> Um exemplo de pesquisa acadêmica, com aplicação prática imediata, usando vetorização de documentos jurídicos:
  <ul>
    <li><b>Autor</b>: <b>Me Amilar Domingos Moreira Martins </b> - (Mestrado Profissional em Administração Pública)</li>
    <li><b>Título</b>: Agrupamento Automático de Documentos Jurídicos com uso de Inteligência Artificial</li>
    <li><b>Ano de Obtenção</b>: 2018</li>
    <li>http://buscatextual.cnpq.br/buscatextual/visualizacv.do?id=K8073990E8</li>
    <li><b>Aplicação do trabalho</b>: em uso no Superior Tribunal de Justiça</li>
</ul>
<p>O experimento do Me Amilar permite identificar peças processuais que tratam de temas semelhantes, mesmo que escrito de formas diferentes e com termos diferentes. A similaridade textual é obtida ao treinar modelos matemáticos com o uso de IA, treinamento não supervisionado. O modelo foi treinado com mais de 300 mil documentos jurídicos do contexto específico do projeto, permitindo que termos próprios do vocabulário jurídico não sejam confundidos com seus significados do senso comum. 
<p>Um dos "segredos" para o sucesso do treinamento é o pré-processamento dos documentos treinados. Com o modelo pronto, novos documentos são pré-processados com o mesmo algoritmo e vetorizados pelo modelo. Os vetores podem ser armazenados em qualquer repositório para comparação futura.
<p>O poder de pesquisa vetorial do <b>MemSQL</b> entra aqui. Armazenando os vetores no MemSQL, junto com outros metadados das peças processuais, por exemplo, pode-se encontrar outras peças processuais semelhantes refinando a pesquisa com metadados (ano, relator, ramo do direito etc), obtendo-se o resultado em segundos ou fraçoes de segundos.
<p>A proposta da <b>pesquisa textual avançada</b> é permitir somar a esse poder de busca vetorial, ou até ser usado de forma independente, uma pesquisa textual com critérios flexíveis e robustos que permitam um resultado rápido e preciso.
<p>Para ilustrar a facilidade do uso do MemSQL na pesquisa vetorial, tem-se o sql abaixo que procura entre milhares de documentos aqueles que estão a uma distância vetorial 0.05 do vetor paradigma. Isso pode ser traduzido grosseiramente em documentos 95% semelhantes ao documento paradigma.
  <ul><li>Select * from base.vetores where DOT_PRODUCT(<b>meu_vetor</b>,vetores.vetor)>=<b>0.95</b></li></ul>


####... em edição
