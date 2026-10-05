# EP 1 - Cálculos Complexos com Séries de Taylor

**Tema:** Cosseno Hiperbólico

**Disciplina:** PROJETO INTEGRADOR: COMPUTAÇÃO CIENTÍFICA

**Integrantes:**

- Vinicius dos Santos Ribeiro
- Nicolas Eugênio
- Bruna Macruz

**Links:**

Google Colab: https://colab.research.google.com/drive/15Bg5Kpnu8JjRUbuHzR9lKkauhaORHhea#scrollTo=iBsVJkh9b0aL

Youtube: https://youtu.be/OpPNlnDI7HY

---

## 1. Visão Geral do Projeto

Este projeto consiste na simulação e otimização do cálculo do **Cosseno Hiperbólico ($y = \cosh(x)$)** em Python utilizando a **Série de Taylor**.

A motivação computacional decorre do fato de que unidades centrais de processamento (CPUs) realizam nativamente apenas operações aritméticas fundamentais (soma, subtração, multiplicação e divisão). Para avaliar funções transcendentais e hiperbólicas sem hardware dedicado, bibliotecas de software recorrem a aproximações polinomiais.

---

## 2. Fundamentação Teórica

### 2.1 Definição da Função

O Cosseno Hiperbólico é definido analiticamente por:
$$\cosh(x) = \frac{e^x + e^{-x}}{2}$$

### 2.2 Derivadas Calculadas Manualmente

Para a construção da Série de Maclaurin (Série de Taylor em torno de $x_0 = 0$), as derivadas sucessivas foram obtidas sem o uso de bibliotecas simbólicas:

1. $f(x) = \cosh(x) \implies f(0) = \cosh(0) = 1$
2. $f'(x) = \sinh(x) \implies f'(0) = \sinh(0) = 0$
3. $f''(x) = \cosh(x) \implies f''(0) = \cosh(0) = 1$
4. $f'''(x) = \sinh(x) \implies f'''(0) = \sinh(0) = 0$
5. $f^{(4)}(x) = \cosh(x) \implies f^{(4)}(0) = \cosh(0) = 1$

**Padrão das derivadas:**

- Para ordens pares ($n = 2k$): $f^{(2k)}(0) = 1$
- Para ordens ímperes ($n = 2k+1$): $f^{(2k+1)}(0) = 0$

### 2.3 Expansão em Série de Taylor

Aplicando o padrão das derivadas na fórmula geral de Taylor:
$$\cosh(x) = \sum_{n=0}^{\infty} \frac{x^{2n}}{(2n)!} = 1 + \frac{x^2}{2!} + \frac{x^4}{4!} + \frac{x^6}{6!} + \dots$$

### 2.4 Justificativa para o Limite de $N$

Adota-se como padrão um limite máximo de **$N = 10$ termos** ($n_{max} = 10$). Devido ao crescimento fatorial do denominador $(2n)!$, o termo genérico $a_n = \frac{x^{2n}}{(2n)!}$ tende rapidamente a zero para valores de $x$ próximos da origem ($\vert{}x\vert{} \le 3$), garantindo convergência e precisão da ordem de $10^{-16}$ (limite do tipo de dado `float64`) com poucos loops.

---

## 3. Otimização do Algoritmo e Tabela de Busca

### 3.1 Recorrência para Otimização

A versão não otimizada recalcula potenciações (`np.pow`) e fatoriais (`math.factorial`) a cada iteração ($O(N^2)$ operações aritméticas).

A **versão otimizada** reaproveita os resultados da iteração anterior através da razão de transição entre termos consecutivos:
$$a_n = a_{n-1} \cdot \frac{x^2}{(2n-1)(2n)}$$

Isso reduz o custo computacional do laço para operações básicas ($O(N)$).

### 3.2 Tabela de Valores Notáveis (Lookup Table)

Para pontos notáveis de uso frequente, utiliza-se a comparação com tolerância de ponto flutuante (`math.isclose`):

| $x$       | Valor Exato / Conhecido                  | Retorno Imediato          |
| :-------- | :--------------------------------------- | :------------------------ |
| $0$       | $1.0$                                    | `1.0`                     |
| $1$       | $\frac{e + e^{-1}}{2} \approx 1.5430806$ | `(math.e + 1/math.e) / 2` |
| $-1$      | $\frac{e + e^{-1}}{2} \approx 1.5430806$ | `(math.e + 1/math.e) / 2` |
| $\ln(2)$  | $1.25$                                   | `1.25`                    |
| $-\ln(2)$ | $1.25$                                   | `1.25`                    |

---

## 4. Instruções de Instalação e Execução

### 4.1 Pré-requisitos e Bibliotecas

O projeto requer Python 3.8+ e as seguintes bibliotecas:

- `numpy`
- `matplotlib`

Instalação via `pip`:

```bash
pip install numpy matplotlib
```

### 4.2 Execução do Software

Você pode executar o projeto de duas formas:

1. **Via Google Colab / Jupyter Notebook:**
   - Abra o arquivo `EP1_PI.ipynb`.
   - Execute as células em ordem sequencial (`Runtime` > `Run all` ou `Ctrl+F9`).

2. **Via Linha de Comando (Terminal):**
   ```bash
   python EP1_Cosseno_Hiperbolico.py
   ```

## 5. Exemplos de Input e Output

### Exemplo 1: Teste isolado com $x = 1.0$ e $N = 5$

**Input (Python):**

```python
x = 1.0
n = 5
print("Numpy:", np.cosh(x))
print("Taylor Otimizado:", cosh_otimizado_com_tabela(x, n))
```

**Output:**

```text
Numpy: 1.5430806348152437
Taylor Otimizado: 1.5430806348152437
```

### Exemplo 2: Consulta na Tabela de Busca ($x = \ln(2)$)

**Input (Python):**

```python
import math
x = math.log(2)
print("Taylor (Tabela):", cosh_otimizado_com_tabela(x))
```

**Output:**

```text
Taylor (Tabela): 1.25
```

---

## 6. Análise de Desempenho (Erro vs. Tempo de Execução)

O script realiza o benchmark com `timeit` avaliando a função em $360$ pontos no intervalo $x \in [-3, 3]$ para $N \in [1, 10]$:

- **Erro Médio vs. $N$:** o erro decresce exponencialmente à medida que $N$ aumenta, atingindo a precisão limite da máquina (`float64`) por volta de $N = 7$.
- **Tempo de Processamento vs. $N$:** o tempo de execução cresce de forma linear simples, na ordem de microssegundos ($\mu s$).
