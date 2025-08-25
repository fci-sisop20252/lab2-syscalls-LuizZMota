# 📝 Relatório do Laboratório 2 - Chamadas de Sistema

---

## 1️⃣ Exercício 1a - Observação printf() vs 1b - write()

### 💻 Comandos executados:
```bash
strace -e write ./ex1a_printf
strace -e write ./ex1b_write
```

### 🔍 Análise

**1. Quantas syscalls write() cada programa gerou?**
- ex1a_printf: 10 syscalls
- ex1b_write: 7 syscalls

**2. Por que há diferença entre os dois métodos? Consulte o docs/printf_vs_write.md**

```
Porque printf usa buffering da libc e só chama write() quando o buffer enche ou ocorre flush, 
podendo gerar mais syscalls. Já write chama o kernel diretamente a cada chamada, sem buffer.

```

**3. Qual método é mais previsível? Por quê você acha isso?**

```
O write é mais previsível, pois cada chamada gera exatamente uma syscall, sem depender de buffers internos.

```

---

## 2️⃣ Exercício 2 - Leitura de Arquivo

### 📊 Resultados da execução:
- File descriptor: 3
- Bytes lidos: 128

### 🔧 Comando strace:
```bash
strace -e openat,read,close ./ex2_leitura
```

### 🔍 Análise

**1. Qual file descriptor foi usado? Por que não começou em 0, 1 ou 2?**

```
Foi usado o descriptor 3, pois 0, 1 e 2 já são reservados para stdin, stdout e stderr.
```

**2. Como você sabe que o arquivo foi lido completamente?**

```
Porque a syscall read() retorna 0, indicando fim de arquivo.

```

**3. Por que verificar retorno de cada syscall?**

```
Para ver se tem erros de execução, tipo falha de abertura, leitura ou escrita, 
e ter certeza que a operação foi realizada na forma correta.

```

---

## 3️⃣ Exercício 3 - Contador com Loop

### 📋 Resultados (BUFFER_SIZE = 64):
- Linhas: 25 (esperado: 25)
- Caracteres: 1300
- Chamadas read(): 21
- Tempo: 0.000091 segundos

### 🧪 Experimentos com buffer:

| Buffer Size | Chamadas read() | Tempo (s) |
|-------------|-----------------|-----------|
| 16          |    82       |    0.000218      |
| 64          |    21             |     0.000091      |
| 256         |      6           | 0.000081          |
| 1024        |     2            | 0.000075          |

### 🔍 Análise

**1. Como o tamanho do buffer afeta o número de syscalls?**

```
Quanto mais Buffers mais chamadas para ler a mesma quantidade de dados.
```

**2. Todas as chamadas read() retornaram BUFFER_SIZE bytes? Discorra brevemente sobre**

```
Não. O read() retorna até BUFFER_SIZE, mas pode devolver menos dependendo da disponibilidade de dados ou fim de arquivo.
```

**3. Qual é a relação entre syscalls e performance?**

```
Mais syscalls significam maior overhead e pior desempenho, e aí menos syscalls reduzem o custo de troca com o kernel e aumentam a performance.
```

---

## 4️⃣ Exercício 4 - Cópia de Arquivo

### 📈 Resultados:
- Bytes copiados: 1364
- Operações: 6
- Tempo: 0.000206 segundos
- Throughput: 6466.17 KB/s

### ✅ Verificação:
```bash
diff dados/origem.txt dados/destino.txt
```
Resultado: [X] Idênticos [ ] Diferentes

### 🔍 Análise

**1. Por que devemos verificar que bytes_escritos == bytes_lidos?**

```
Porque o `write()` pode escrever menos bytes do que foi pedido. Se não ver isso, pode ter o risco de perder dados nessa transferência.

```

**2. Que flags são essenciais no open() do destino?**

```
- `O_WRONLY`: abre o arquivo para escrita.
- `O_CREAT`: cria o arquivo se ele não existir.
- `O_TRUNC`: trunca (zera) o conteúdo do arquivo se ele já existir, garantindo que não reste lixo de versões anteriores.

```

**3. O número de reads e writes é igual? Por quê?**

```
Cada read() bem-sucedido é seguido por um write(). Porém, em casos especiais (como interrupções ou escrita parcial), pode ser necessário fazer múltiplos write() para um único read().

```

**4. Como você saberia se o disco ficou cheio?**

```
A syscall write() retornaria um valor menor que o esperado ou -1, e `errno` seria definido com o erro `ENOSPC`.

```

**5. O que acontece se esquecer de fechar os arquivos?**

```
Recursos do sistema podem não ser liberados imediatamente, podendo causar vazamentos de recursos. Pode haver perda de dados que estavam no buffer, pois o kernel pode postergar a escrita física até o `close()` ser chamado. Em sistemas maiores, isso pode esgotar o limite de descritores por processo, impedindo a abertura de novos arquivos.

```

---

## 🎯 Análise Geral

### 📖 Conceitos Fundamentais

**1. Como as syscalls demonstram a transição usuário → kernel?**

```
Syscalls como read(), write(), open(), close() são interfaces fornecidas pela biblioteca do sistema para interagir com o kernel. Quando chamadas, há uma transição de "modo usuário" para "modo kernel", onde o sistema operacional tem permissões totais para acessar o hardware (como disco, memória etc.). Isso é essencial para a segurança e estabilidade do sistema.

```

**2. Qual é o seu entendimento sobre a importância dos file descriptors?**

```
File descriptors são inteiros que representam arquivos abertos no contexto de um processo. Eles atuam como "ponteiros" para tabelas internas do kernel que contêm metadados e estados dos arquivos. Tudo que envolve entrada e saída é feito via descriptors, o que torna o sistema unificado e flexível.
```

**3. Discorra sobre a relação entre o tamanho do buffer e performance:**

```
Buffers maiores geralmente resultam em menos chamadas read() e write(), reduzindo o overhead de transição usuário-kernel, e aumentando a performance. Mas, buffers muito grandes consomem mais memória. 

```

### ⚡ Comparação de Performance

```bash
# Teste seu programa vs cp do sistema
time ./ex4_copia
time cp dados/origem.txt dados/destino_cp.txt
```

**Qual foi mais rápido?** ex4_copia

**Por que você acha que foi mais rápido?**

```
O comando `cp` é altamente otimizado e utiliza técnicas como buffers grandes, chamadas assíncronas, mapeamento de memória (mmap), ou até operações específicas do kernel (como `sendfile()`) para copiar arquivos com eficiência máxima. Nosso programa usa um buffer pequeno (256 bytes) e operações básicas, o que torna a cópia mais lenta.

```

---

## 📤 Entrega
Certifique-se de ter:
- [ ] Todos os códigos com TODOs completados
- [ ] Traces salvos em `traces/`
- [ ] Este relatório preenchido como `RELATORIO.md`

```bash
strace -e write -o traces/ex1a_trace.txt ./ex1a_printf
strace -e write -o traces/ex1b_trace.txt ./ex1b_write
strace -o traces/ex2_trace.txt ./ex2_leitura
strace -c -o traces/ex3_stats.txt ./ex3_contador
strace -o traces/ex4_trace.txt ./ex4_copia
```
# Bom trabalho!
