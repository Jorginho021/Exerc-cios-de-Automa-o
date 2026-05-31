# Exerc-cios-de-Automa-o


Exercício 1 — O Caos no Servidor da Escola

#!/bin/bash

# Diretório principal
DIR="/home/alunos"

# Criando subpastas
mkdir -p "$DIR/documentos"
mkdir -p "$DIR/imagens"
mkdir -p "$DIR/videos"
mkdir -p "$DIR/outros"

# Contadores
doc=0
img=0
vid=0
out=0
processados=0

# Espaço antes da limpeza
antes=$(du -sb "$DIR" | cut -f1)

echo "Iniciando organização..."

# Percorre os arquivos
for arquivo in "$DIR"/*; do

    # Verifica se é arquivo
    [ -f "$arquivo" ] || continue

    nome=$(basename "$arquivo")

    case "$arquivo" in

        *.txt|*.pdf)
            mv "$arquivo" "$DIR/documentos/"
            ((doc++))
            ;;

        *.jpg|*.jpeg|*.png)
            mv "$arquivo" "$DIR/imagens/"
            ((img++))
            ;;

        *.mp4|*.avi|*.mkv)
            mv "$arquivo" "$DIR/videos/"
            ((vid++))
            ;;

        *.tmp)
            rm -f "$arquivo"
            ;;

        *)
            mv "$arquivo" "$DIR/outros/"
            ((out++))
            ;;
    esac

    ((processados++))

    # Exibe progresso a cada 100 arquivos
    if (( processados % 100 == 0 )); then
        echo "$processados arquivos processados..."
    fi

done

# Espaço depois da limpeza
depois=$(du -sb "$DIR" | cut -f1)

# Calcula espaço liberado
liberado=$((antes - depois))

# Data atual
data=$(date)

# Relatório
relatorio="$DIR/relatorio_organizacao.txt"

echo "RELATÓRIO DE ORGANIZAÇÃO" > "$relatorio"
echo "Data da execução: $data" >> "$relatorio"
echo "" >> "$relatorio"
echo "Arquivos movidos:" >> "$relatorio"
echo "Documentos: $doc" >> "$relatorio"
echo "Imagens: $img" >> "$relatorio"
echo "Vídeos: $vid" >> "$relatorio"
echo "Outros: $out" >> "$relatorio"
echo "" >> "$relatorio"
echo "Espaço liberado: $liberado bytes" >> "$relatorio"

echo "Organização concluída!"
echo "Relatório salvo em: $relatorio"



Exercício 2 — Backup Noturno que Nunca Acontece


#!/bin/bash

ORIGEM="/clinica/dados"
DESTINO="/mnt/hd_externo/backups"
DATA=$(date +%F)
ARQUIVO="backup_$DATA.tar.gz"
LOG="/var/log/backup_clinica.log"

mkdir -p "$DESTINO"

tar -czf "$DESTINO/$ARQUIVO" "$ORIGEM"

find "$DESTINO" -name "*.tar.gz" -mtime +30 -delete

TAMANHO=$(du -h "$DESTINO/$ARQUIVO" | cut -f1)

echo "$(date) | Backup: $ARQUIVO | Tamanho: $TAMANHO | Status: OK" >> "$LOG"

echo 
echo "Backup concluído"
echo "Arquivo: $ARQUIVO"
echo "Tamanho: $TAMANHO"
echo "Destino: $DESTINO"
echo 

Exercício 3 — 500 Alunos para Cadastrar
#!/bin/bash

CSV="alunos.csv"
RELATORIO="relatorio_cadastro.txt"

criados=0
falhas=0

echo "RELATÓRIO DE CADASTRO" > "$RELATORIO"
echo "=====================" >> "$RELATORIO"

tail -n +2 "$CSV" | while IFS=',' read nome cpf turma email
do

usuario=$(echo "$nome" | awk '{print tolower($1)}')

mkdir -p "/turmas/$turma"

if id "$usuario" &>/dev/null
then
    echo "Usuário já existe: $usuario" >> "$RELATORIO"
else
    useradd -m "$usuario"

    senha=$(echo "$cpf" | tr -d '.-')

    echo "$usuario:$senha" | chpasswd

    echo "$usuario" >> "/turmas/$turma/alunos.txt"

    echo "$usuario - criado" >> "$RELATORIO"
fi

done


Exercício 4 — Logs que Ninguém Lê

#!/bin/bash

DATA=$(date -d "yesterday" +%F)

LOG="/var/log/webserver/$DATA.log"

mkdir -p /relatorios

REL="/relatorios/log_$DATA.txt"

echo "RELATÓRIO DE LOGS" > "$REL"
echo "Data: $DATA" >> "$REL"
echo "" >> "$REL"

echo "Total 200: $(grep -c '| 200 |' "$LOG")" >> "$REL"
echo "Total 404: $(grep -c '| 404 |' "$LOG")" >> "$REL"
echo "Total 500: $(grep -c '| 500 |' "$LOG")" >> "$REL"

echo "" >> "$REL"
echo "TOP 5 ERROS 404" >> "$REL"

grep "| 404 |" "$LOG" | awk -F'|' '{print $3}' | sort | uniq -c | sort -nr | head -5 >> "$REL"

echo "" >> "$REL"
echo "REQUISIÇÕES ACIMA DE 2s" >> "$REL"

awk -F'|' '
{
tempo=$4
gsub("s","",tempo)
if(tempo > 2)
print $0
}
' "$LOG" >> "$REL"

ERROS500=$(grep -c '| 500 |' "$LOG")

if [ "$ERROS500" -gt 10 ]
then
echo "ALERTA: MAIS DE 10 ERROS 500 DETECTADOS!"
fi

echo "Relatório salvo em $REL"


Exercício 5 — Servidor Que Ninguém Monitora

#!/bin/bash

LOG="/var/log/monitor_servidor.log"

DATA=$(date "+%Y-%m-%d %H:%M:%S")

CPU=$(top -bn1 | grep "Cpu(s)" | awk '{print int($2)}')

RAM=$(free | awk '/Mem:/ {printf("%.0f"), $3/$2 * 100}')

DISCO=$(df / | awk 'NR==2 {gsub("%",""); print $5}')

STATUS="[OK]"

if [ "$CPU" -gt 80 ] || [ "$RAM" -gt 85 ] || [ "$DISCO" -gt 90 ]
then
STATUS="[ALERTA]"
fi

if pgrep sshd > /dev/null
then
PROC="sshd OK"
else
PROC="sshd FALHOU"
fi

echo "$DATA $STATUS CPU=${CPU}% RAM=${RAM}% DISCO=${DISCO}% PROC=$PROC" >> "$LOG"


⭐ Exercício para Avaliação — A Virada de Mês que Travava a Empresa

#!/bin/bash

PENDENTES="/dados/filiais/pendentes"
RELATORIO="/dados/relatorios/fechamento_mensal.txt"
ARQUIVO="/dados/arquivo"
LOG="/var/log/fechamento.log"

MES=$(date +%B)
ANO=$(date +%Y)

mkdir -p "$ARQUIVO/$ANO/$MES"
mkdir -p "/dados/relatorios"

TOTAL=0
ARQUIVOS=0

echo "RELATÓRIO DE FECHAMENTO" > "$RELATORIO"
echo "Data: $(date)" >> "$RELATORIO"
echo "" >> "$RELATORIO"

for arquivo in "$PENDENTES"/*.txt
do

[ -f "$arquivo" ] || continue

FILIAL=$(grep "FILIAL:" "$arquivo" | cut -d':' -f2)

VALOR=$(grep "TOTAL:" "$arquivo" | \
sed 's/.*R\$ //' | \
tr -d '.' | \
sed 's/,/./')

TOTAL=$(echo "$TOTAL + $VALOR" | bc)

echo "$FILIAL - R\$ $VALOR" >> "$RELATORIO"

echo "Processando:$FILIAL"

mv "$arquivo" "$ARQUIVO/$ANO/$MES"

((ARQUIVOS++))

done

echo "" >> "$RELATORIO"
echo "TOTAL GERAL: R\$ $TOTAL" >> "$RELATORIO"
echo "" >> "$RELATORIO"
echo "Relatório gerado automaticamente em $(date)" >> "$RELATORIO"

echo "$(date '+%Y-%m-%d %H:%M:%S') FECHAMENTO EXECUTADO - Arquivos:$ARQUIVOS - Total:R\$ $TOTAL" >> "$LOG"

echo 
echo "FECHAMENTO CONCLUÍDO"
echo "Arquivos processados: $ARQUIVOS"
echo "Total consolidado: R\$ $TOTAL"
echo "Relatório salvo em: $RELATORIO"
echo "Log salvo em: $LOG"
echo 


