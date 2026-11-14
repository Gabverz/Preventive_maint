# Dicionário de dados: paradas de uma fresadora

A base registra o histórico de produção de uma fresadora industrial. Cada linha é uma peça usinada: as leituras dos sensores naquele ciclo e se a máquina parou.

| | |
|---|---|
| Linhas | 9.203 |
| Colunas | 14 |
| Paradas | 314 (3,41%) |
| Valores ausentes | 0 |

Arquivo: `data/dados_aula.csv`. As linhas estão na ordem em que as peças foram produzidas.

## Sequência e identificador

| Coluna | Tipo | Significado |
|---|---|---|
| `sequencia_peca` | inteira | Posição da peça na sequência de produção, de 1 a 9.203. Indica o tempo: a peça 1 foi a primeira, a 9.203 a última. Serve para dividir os dados em treino e teste respeitando o tempo. Não deve ser usada como variável do modelo. |
| `codigo_peca` | texto | Código único da peça: uma letra seguida de seis dígitos. A letra repete a classe da peça. Não deve ser usado como variável do modelo. |

## Variáveis de entrada

São as seis leituras disponíveis antes de saber se a máquina parou. É com elas que o modelo deve ser treinado.

| Coluna | Tipo | Unidade | Faixa observada | Significado |
|---|---|---|---|---|
| `classe_peca` | categórica | | 3 categorias | Qualidade da peça produzida. Ver tabela de categorias abaixo. |
| `temp_entrada_c` | decimal | °C | 22,2 a 31,4 | Temperatura do ar na entrada da máquina. |
| `temp_processo_c` | decimal | °C | 32,6 a 40,7 | Temperatura do processo de usinagem. Fica cerca de 10 °C acima da temperatura de entrada. |
| `velocidade_rpm` | inteira | rpm | 1.166 a 2.883 | Velocidade de rotação do eixo. |
| `torque_nm` | decimal | Nm | 3,8 a 76,6 | Torque aplicado pela máquina na peça. Média perto de 40 Nm. |
| `uso_ferramenta_min` | inteira | minutos | 0 a 252 | Tempo acumulado de uso da ferramenta de corte. Mede o desgaste: quanto maior, mais gasta está a ferramenta. |

Categorias de `classe_peca`:

| Valor | Letra no código da peça | Participação |
|---|---|---|
| `economica` | `E` | 60,1% |
| `padrao` | `P` | 29,9% |
| `premium` | `R` | 10,0% |

## Variável alvo

| Coluna | Tipo | Significado |
|---|---|---|
| `parada` | binária (0 ou 1) | 1 se a máquina parou durante a usinagem da peça, 0 se não parou. É o que o modelo deve prever. Apenas 3,41% das linhas têm valor 1. |

## Modos de parada

Cinco colunas binárias que dizem por qual motivo a máquina parou.

**Estas colunas são preenchidas por um operador, depois da parada.** Quando a máquina para, o operador avalia o que aconteceu e registra o modo, como um laudo. Elas não vêm de sensor e não existem no momento em que a peça está sendo usinada.

Por isso não podem entrar como variável de entrada do modelo: no momento da previsão, a informação ainda não existe. Usar qualquer uma delas é vazamento de alvo.

| Coluna | Casos | Significado |
|---|---|---|
| `modo_a` | 91 | Sobre-esforço: combinação de ferramenta muito gasta com torque alto. O limite depende da classe da peça. |
| `modo_b` | 43 | Desgaste de ferramenta: a ferramenta chegou ao fim da vida útil. |
| `modo_c` | 107 | Dissipação de calor: pouca diferença entre a temperatura do processo e a de entrada, junto com rotação baixa. |
| `modo_d` | 88 | Potência: a potência do processo (torque multiplicado pela velocidade de rotação) ficou fora da faixa de operação, para cima ou para baixo. |
| `modo_e` | 18 | Aleatório: parada sem relação com as leituras dos sensores. |

Observações sobre os modos:

- Uma parada pode ter mais de um modo marcado: isso ocorre em 23 linhas.
- Há 8 linhas com `parada = 1` e nenhum modo marcado.
- Há 17 linhas com algum modo marcado e `parada = 0`.

## Papel de cada coluna no modelo

| Papel | Colunas |
|---|---|
| Sequência no tempo, usada só para dividir os dados | `sequencia_peca` |
| Identificador, fora do modelo | `codigo_peca` |
| Variáveis de entrada | `classe_peca`, `temp_entrada_c`, `temp_processo_c`, `velocidade_rpm`, `torque_nm`, `uso_ferramenta_min` |
| Alvo | `parada` |
| Laudo preenchido pelo operador após a parada, fora do modelo | `modo_a`, `modo_b`, `modo_c`, `modo_d`, `modo_e` |
