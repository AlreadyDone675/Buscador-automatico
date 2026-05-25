Buscador de Gravações — HDD Edition

Sistema profissional de busca massiva de arquivos otimizado para HDD mecânico, projetado para trabalhar com milhões de arquivos sem travar a interface.
Baseado em arquitetura HDD-first, priorizando throughput sequencial, baixo overhead e estabilidade.

🚀 Features
Busca em 5M+ arquivos
Compatível com:
.mp3
.wav
.ogg
Busca por números contidos no nome do arquivo
Interface gráfica com Tkinter
Leitura de números via Excel/CSV
Sistema otimizado para HDD mecânico
Scanner extremamente leve usando os.scandir()
Match otimizado com:
pyahocorasick (recomendado)
fallback para regex compilada
Thread única de I/O para evitar gargalo de seeks
Interface passiva (sem travamentos)
Sistema de parada segura
Logs de erro
Cópia automática dos arquivos encontrados
🧠 Filosofia do Projeto

Este projeto NÃO tenta usar:

multithreading agressivo
milhares de callbacks Tkinter
filas complexas
polling excessivo
renderização constante da UI

Porque em HDD mecânico isso normalmente:

piora performance
aumenta seeks
reduz throughput
cria gargalo de CPU/GIL

O foco aqui é:

Máximo throughput sequencial do disco
⚡ Performance Esperada

Hardware alvo:

Intel i7 5ª geração
16 GB RAM
HDD mecânico (~112 MB/s)

Velocidade esperada:

Cenário	Velocidade
HDD comum	3k ~ 8k arquivos/s
SSD SATA	10k ~ 30k arquivos/s
NVMe	Muito acima disso
📂 Estrutura do Sistema
Projeto/
│
├── buscador.py
├── numeros.xlsx
├── Encontrados/
│
└── (diretório externo gigantesco onde os arquivos serão procurados)

Fluxo:

Programa inicia
Lê planilha Excel/CSV
Carrega números
Percorre diretório externo
Procura arquivos contendo os números
Copia arquivos encontrados para Encontrados
🔍 Como Funciona
Scanner

Utiliza:

os.scandir()

em vez de:

os.walk()

Porque:

reduz alocações
reduz overhead Python
melhora desempenho em milhões de arquivos
Match de Nomes
Melhor opção (recomendada)

Instale:

pip install pyahocorasick

O sistema utilizará:

algoritmo Aho-Corasick em C
complexidade O(len(nome))
muito mais rápido que regex gigante
Fallback automático

Se pyahocorasick não estiver instalado:

o sistema usa regex compilada
ainda funciona normalmente
porém mais lento
🖥 Interface

A UI foi desenhada para ser:

leve
passiva
não bloquear o scanner

Atualizações:

apenas 1x por segundo
sem redraw massivo
sem Listbox sendo atualizada continuamente
📦 Instalação
1. Clone o projeto
git clone https://github.com/seuusuario/buscador-gravacoes.git
cd buscador-gravacoes
2. Instale as dependências
pip install pandas openpyxl

Opcional (RECOMENDADO):

pip install pyahocorasick
▶ Como Executar
python buscador.py
📑 Formato da Planilha

A planilha deve conter os números na:

Coluna A

Exemplo:

A
11999998888
21988887777
31977776666

O sistema:

remove caracteres especiais
remove .0
limpa espaços automaticamente
⚙ Configurações

No topo do código:

EXTENSOES = frozenset({".mp3", ".wav", ".ogg"})
Atualização da UI
UI_INTERVALO_S = 1.0
Máximo de linhas exibidas
MAX_LINHAS_UI = 5000
🧩 Estratégia HDD-First
Por que NÃO usar múltiplas threads de I/O?

Em HDD:

leitura e escrita simultânea causam seeks constantes
o cabeçote do disco alterna entre posições
throughput despenca

Por isso:

o scanner roda sozinho
a cópia ocorre em sequência
o HDD trabalha de forma mais linear possível
📈 Complexidade
Operação	Complexidade
Varredura	O(N)
Match Aho-Corasick	O(len(nome))
Lookup	O(1)
UI	O(1) por segundo
🛠 Tecnologias
Python 3.11+
Tkinter
pandas
openpyxl
pyahocorasick
os.scandir()
📋 Logs

Erros são gravados em:

buscador.log

O sistema evita logs excessivos para não degradar performance.

📌 Recomendações
HDD mecânico

✅ Recomendado:

Thread única de I/O
Destino no mesmo disco
Sem antivírus analisando os arquivos
Windows Defender excluindo a pasta

❌ Evite:

Muitas threads
Atualização constante da UI
Logs excessivos
Regex gigantescas
Queue excessiva
🔥 Melhor Performance Possível

Para máxima velocidade:

Instale:
pip install pyahocorasick

Isso é o que mais impacta performance no projeto.

📄 Licença
╔══════════════════════════════════════════════════════════════════════════════╗
║   BUSCADOR DE GRAVAÇÕES — ARQUITETURA HDD-FIRST                            ║
║   Princípio: um thread de I/O, match mínimo, UI passiva                    ║
╚══════════════════════════════════════════════════════════════════════════════╝

FILOSOFIA DE DESIGN:
  O gargalo é 100% I/O (HDD mecânico ~100 IOPS aleatório).
  Qualquer overhead de CPU/threading que force seeks adicionais
  é mais caro que o trabalho que substitui.

  Regra de ouro para HDD:
    • UM thread acessa o disco por vez
    • Scanner e copier nunca rodam ao mesmo tempo no mesmo disco
    • Match deve ser O(1) ou O(len(nome)) em C puro, sem Python-loop
    • UI nunca chama nada pesado no caminho crítico

ARQUITETURA:
  Thread Principal (Tkinter)
      └─▶ Worker Thread ÚNICO
              ├─ os.scandir recursivo (stack explícita, BFS)
              ├─ match via pyahocorasick (Aho-Corasick em C)
              │    └─▶ fallback: frozenset + str.__contains__ se sem lib
              ├─ achados acumulados em lista simples (sem Queue)
              └─ cópia serial APÓS varredura (evita seek concorrente)

  UI: root.after(1000) — um timer por segundo, sem polling agressivo
