# EM EDIÇÃO 

# Pesquisa Textual
Pesquisa textual implementada com recursos nativos do banco de dados MemSQL com o uso de dicionário de sinônimos, fonética e aproximações. É uma pesquisa que tenta ir além do que pesquisas comuns fazem, pois não tem o objetivo de trazer grandes volumes de resultados, mas resultados o mais próximo possível do desejado.

### MemSQL
O MemSQL é um banco de dados SQL distribuído e altamente escalável, prometendo desempenho máximo para cargas de trabalho transacionais e analíticas de bigdata, usando modelos de dados bem conhecidos. Suporta metadados JSON, comparação de vetores multidimensionais e análise de grandes volumes de dados. Está disponível em https://www.memsql.com, pode ser usado na versão Community ou com suporte pago. Para facilitar o uso desse banco de dados, o acesso é feito por meio de conectores compatíveis com MySQL, apesar do banco de dados não ter qualquer relação de desenvolvimento com o MySQL, como explica o fabricante. 

### Pesquisa textual padrão do MemSQL
O MemSQL possui uma pesquisa textual nativa, com operadores e curingas com performance considerável em milhões de registros textuais. Possui também a capacidade de pesquisa com regex com performance também considerável em volumes de milhões de registros. <b>Apesar</b> dessa capacidade de pesquisa, que assemelha-se ao <b>ElasticSearch</b> em alguns casos, ele não permite filtros mais refinados nos seus critérios de pesquisa nativos. 
<p>RLIKE: https://docs.memsql.com/sql-reference/v6.8/rlike_and_regexp/ 
<p>MATCH: https://docs.memsql.com/sql-reference/v6.8/match/ 
#### Cursos oficiais disponíveis
<ul><li>https://training.memsql.com/</li></ul>

### Pesquisa textual avançada
Implementei aqui um conceito de pesquisa por proximidade dos termos e outros operadores para refinamento de pesquisa. Esse tipo de operador torna-se importante para refinar pesquisas em grande volume de dados, onde não é iportante trazer muito resultado, mas um resultado o mais próximo possível do que é procurado. Ferramentas comuns de busca como <b>ElasticSearch</b> e o próprio <b>MemSQL</b> não trazem nativamente esse tipo de operador. 
<p> Esse tipo de pesquisa permite o uso de dicionário de sinônimos em qualquer língua, inclusive o uso de fonética. Para evitar não encontrar termos com grafia incorreta de acentos ou termos no singular/plural, bem como números com pontuação ou sem, o texto de entrada é pré-processado. Por padrão o texto é pesquisado no singular, removendo pronomes oblíquos, mas é possível localizar o termo real usando aspas (exceto acentos que sempre são desconsiderados).
<p> O pré-processamento envolve:
<ul>
  <li> retirada de <b>acentos</b> </li>
  <li> <b>singularização</b> rápida: não é um porquguês perfeito, mas uma singularização para a máquina localizar textos </li>
  <li> remoção de <b>pronomes oblíquos</b>: o pré-processamento transforma "fazer-lhe-ia" em "fazer", mas a pesquisa entre aspas pode localizar o termo real "fazer-lhe-ia"</li>
</ul>

<p> A pesquisa permite localizar termos pelo dicionário de sinônimos. Ao pesquisar a palavra "genitor", o sistema pesquisa também "pai". A tabela de sinônimos é flexível, permitindo incluir termos em outras línguas se desejado. O uso de sinônimos pode ser ativado ou desativado a cada pesquisa. Ao pesquisar termos entre aspas, o sinônimo é desativado para o termo ou conjunto de termos entre aspas.
<ul>
  <li> retirada de <b>acentos</b> </li>
  <li> <b>singularização</b> rápida: não é um porquguês perfeito, mas uma singularização para a máquina localizar textos </li>
  <li> remoção de <b>pronomes oblíquos</b>: o pré-processamento transforma "fazer-lhe-ia" em "fazer", mas a pesquisa entre aspas pode localizar o termo real "fazer-lhe-ia"</li>
</ul>

#### Conectores
<ul>
  <li> <b>E</b>: conector padrão, exige a existência do termo no documento</li>
  <li> <b>NÃO</b>: nega a existância de um termo no docuemento </li>
  <li> <b>OU</b> entre termos: indica que um ou outro termo podem ser encontrados para satisfazer a pesquisa ex.: <u>"fazer" OU "feito" E casa</u> realiza uma pesquisa que encontre (fazer ou feito literalmente) e também (casa ou termos que no singular sejam escritos como casa)</li>
  <li> <b>OU</b> com parênteses: permite realizar pesquisas mais complexas. No momento nem todas as combinações de parênteses em pesquisas com OU são possíveis devido ao limite de aninhamentos usados no MemSQL. Ex.: <u>(casa ADJ5 papel) ou (casa ADJ5 moeda)</u>. Nesse caso a pesquisa pode ser simplificada como <u>casa ADJ5 papel ou moeda</u></li>
  <li> <b>ADJ</b>n: permite localizar termos que estejam até n termos a frente do primeiro termo. Ex.: <u>casa ADJ3 papel</u> vai localiar textos que contenham "casa de papel", "casa papel", "casa feita de papel", mas não localizaria "casa feita de muito papel". </li>
  <li> <b>ADJC</b>n: equivalente ao <b>ADJ</b> padrão, mas obriga que os dois termos estejam presentes no mesmo parágrafo. Não necessariamente a mesma sentença, mas o mesmo parágrafo considerando a quebra /n ou <br> no texto </li>
  <li> <b>PROX</b>n: </li> semelhante ao <b>ADJ</b>, mas localiza termos posteriores ou anteriores ao primeiro termo pesquisado. Ex.: <u>casa PROX3 papel</u> vai localiar textos que contenham "casa de papel", "papel na casa", "papel colado na casa", "casa feita de papel", mas não localizaria "casa feita de muito papel" ou "papel desenhado e colado na casa".
  <li> <b>PROXC</b>n: equivalente ao <b>PROX</b> padrão, mas obriga que os dois termos estejam presentes no mesmo parágrafo. Não necessariamente a mesma sentença, mas o mesmo parágrafo considerando a quebra /n ou <br> no texto </li>
  <li> <b>COM</b>: obriga que os dois termos pesquisados estejam presentes em um mesmo parágrafo, independente da distância e da ordem. Ex.: <u>casa COM papel</u> localiza todos os textos que contenham a palavra casa e a palavra papel, em qualquer ordem e distência, mas que estejam em um mesmo parágrafo. </li>
  <li> <b>MESMO</b>: os documentos podem ser indexados com um tipo único, ou com tipos independentes como um resumo e o texto original. O operador MESMO permite que o documento seja encontrado apenas se os termos pesquisados estejam em um mesmo tipo do documento. Sem o operador MESMO, o texto será localizado se tiver um termo em um tipo (resumo por exemplo) e outro termo em outro tipo (índice remissivo, por exemplo).  </li>  
</ul>

### Similaridade semântica
Somando-se essa pesquisa refinada ao poder de busca vetorial do MemSQL, podemos ter um sistema de pesquisa avançado que consegue pesquisar em poucos segundos textos semelhantes a um texto paradigma ou textos que contenham determinados critérios refinados. Bem como unir a pesquisa vetorial à pesquisa textual acançada.
<p>A pesquisa vetorial é nativa do MemSQL, sendo necessário apenas treinar ou usar um modelo já treinado para vetorizar textos. Isso pode ser feito de forma relativamente simples usando frameworks como o DOC2VEC do Gensim. A busca vetorial do MemSQL promete, e cumpre, a comparação vetorial em um volume considerável de dados (dezenas de milhões de vetores) em segundos ou até frações de segundos.


... em edição
