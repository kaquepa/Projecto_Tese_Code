## Ataque ao criptosistema de McEliece — Implementação(Python + Sage)
# Resumo
Este repositório contém uma implementação em Python (com dependências do SageMath) de um script que gera um código de Goppa aleatório, constrói uma chave pública no estilo McEliece e tenta recuperar o vetor de erro usando três estratégias de information set decoding (McEliece, Lee-Brickell e Leon). O objetivo é servir como código-educacional para estudar ataques por information set decoding contra o esquema McEliece.

O código depende fortemente de funcionalidades do SageMath (rings finitos, construção de GoppaCode, etc.).

# Requisitos
SageMath (recomendado: SageMath 9.x ou 10.x) — o script importa módulos de sage.* para campos finitos, polinômios e GoppaCode.

Python 3.8+ (o interpretador do Sage normalmente já fornece um Python compatível).
Pacotes Python: numpy, sympy. (Novamente: o ambiente Sage costuma já trazer numpy e sympy.)

- Como executar
Abra um terminal com o ambiente do Sage ativado (por exemplo, usando sage -sh ou executando sage interativamente dependendo da sua instalação).

Salve o script em um ficheiro attack.py (ou outro nome à sua escolha).
Execute-o com o Python do Sage. Exemplos:
# opção A: executar diretamente no shell do Sage
sage -python attack.py

# opção B: iniciar REPL do Sage e executar
sage
# $ exec(open('attack.py').read())
Parâmetros principais (no final do script)
O script configura e executa um ataque com a linha:
- m, t, p, l = 5, 4, 3, 3
- attack = Attack(m, t, p, l)
attack.main()
. m — extensão do corpo do campo binário; o comprimento do código é n = 2^m.
. t — grau do polinômio de Goppa e número máximo de erros corrigíveis (parâmetro de correção).
. p — parâmetro usado nas variantes Lee-Brickell/Leon: número máximo de posições de suporte consideradas em algumas buscas combinatórias.

l — parâmetro usado no algoritmo de Leon para dividir colunas não-pivô em duas partes (ver o código para detalhes).
A dimensão k do código é calculada como k = n - m * t (assumindo que o código gerado tenha dimensão esperada).

# O que o script faz — visão geral
Geração de código Goppa aleatório: usa construções do Sage para escolher um polinómio irreducível g(x) de grau t e um conjunto de pontos L para construir GoppaCode(g, L).
Gerar chave pública: obtém a matriz geradora do código (G), aplica uma matriz invertível S e uma permutação P para formar uma chave pública G' = S * G * P.
Codificação e introdução de erro: gera uma mensagem aleatória, computa o código e aplica um vetor de erro com t posições aleatórias para gerar ciphertext.

# Ataques (information set decoding): tenta encontrar o vetor de erro com três variantes:
- DecodeError_McEliece_information (variante básica de information set decoding);
- DecodeError_LeeBrickell (variante de Lee–Brickell que tenta adicionar vetores esparsos sobre o conjunto informação);
- DecodeError_Leon (variante de Leon que divide as colunas não-pivô em duas partes para acelerar a busca).
Verificação/recuperação da mensagem: ao recuperar (parcialmente) o erro, reconstrói o codeword e tenta recuperar a mensagem original.

# Saída esperada
O script imprime mensagens intermédias, por exemplo:
Quantidade de tentativas para gerar o polinômio de Goppa adequado.
O information set encontrado por cada algoritmo e o número de iterações gastas.
O ciphertext, o vetor de erro encontrado por (pelo menos) uma das heurísticas, e se o codeword recuperado é idêntico ao original.

Mensagens finais indicando sucesso ou Attack failed!! se a recuperação falhar.

Exemplo de saída (formatada):
1 trials untils  generated

```bash 
information_set McEliece: [..]    iteration: 25
information_set brickell: [..]    iteration: 1
information_set leon: [..]       iteration: 5
ciphertext [ ... ]
error using LeeBrickell [ ... ]

Os codeword não sao identicos
Attack failed!!
```
# Arquitetura e pontos importantes do código
A classe Attack contém todas as rotinas: geração de código, formação de chave pública, operações matriciais sobre GF(2), e as três rotinas de decodificação/ataque.
A conversão entre estruturas do Sage (matrizes) e numpy é feita em vários pontos; atenção a tipos e operações módulo 2 (np.mod, np.bitwise_xor).

- A função echelon_form usa sympy.Matrix.rref() seguida de redução módulo 2 — esse passo é sensível a representação fracionária interna do rref e por isso há transformação racional → inteiro módulo 2.
- A geração da matriz S ignorava o método clássico de escolher uma matriz invertível sobre GF(2); o script usa np.random.permutation e testa o determinante numa conversão float. Isto é frágil (pode falhar para determinados tamanhos) e não é o método ideal — ver seção de melhorias.

- Melhorias sugeridas / Observações
Gerar matriz S de forma correta sobre GF(2): em vez de usar permutações e np.linalg.det em float, gerar uma matriz aleatória binária e verificar invertibilidade usando operações em GF(2) (por exemplo sympy.Matrix(S).det_mod(2) ou Matrix(S).inv_mod(2) com captura de exceções).

- Uso consistente de tipos: reduzir a conversão constante entre numpy, sympy e objetos do Sage. Preferir trabalhar com um único tipo para cada rotina (por ex. usar sympy para eliminação Gaussiana e inversões mod 2).
Paralelização: as tentativas aleatórias (varias configurações de conjuntos de informação) podem ser paralelizadas para acelerar testes empíricos.

- Registro / logging: adicionar logging em vez de print para controlar verbosidade.
- Testes e reproducibilidade: permitir uma seed para o RNG para reproduzir execuções.
