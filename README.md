# WiFi7
Guia Completo: Simulação WiFi 7 no ns-3 (WSL Ubuntu)
📋 Sumário

Preparação do Ambiente WSL
Instalação do ns-3
Compilação e Execução
Análise dos Resultados
Experimentos Sugeridos


1. Preparação do Ambiente WSL
1.1 Atualizar o Sistema
bash# Atualizar repositórios e pacotes
sudo apt update && sudo apt upgrade -y
Por quê? Garantir que temos as versões mais recentes das bibliotecas e ferramentas.
1.2 Instalar Dependências do ns-3
bash# Compiladores e ferramentas de build
sudo apt install -y build-essential gcc g++ cmake ninja-build

# Python e ferramentas relacionadas
sudo apt install -y python3 python3-pip python3-dev

# Bibliotecas necessárias para ns-3
sudo apt install -y git mercurial wget
sudo apt install -y libsqlite3-dev libxml2 libxml2-dev
sudo apt install -y libgtk-3-dev qtbase5-dev qtchooser qt5-qmake qtbase5-dev-tools
sudo apt install -y gir1.2-goocanvas-2.0 python3-gi python3-gi-cairo python3-pygraphviz gir1.2-gtk-3.0 ipython3
sudo apt install -y openmpi-bin openmpi-common openmpi-doc libopenmpi-dev
sudo apt install -y gdb valgrind
sudo apt install -y doxygen graphviz imagemagick
sudo apt install -y texlive texlive-extra-utils texlive-latex-extra texlive-font-utils dvipng latexmk
Explicação das dependências:

build-essential, gcc, g++: Compiladores C/C++ necessários
cmake, ninja-build: Sistemas de build modernos
python3: ns-3 usa Python para scripts e bindings
libsqlite3-dev: Armazenamento de dados de simulação
libxml2: Parsing de arquivos de configuração
libgtk/qt: Interfaces gráficas (opcional, mas útil)
openmpi: Simulações paralelas distribuídas
gdb, valgrind: Debugging e análise de memória
doxygen, graphviz: Geração de documentação

1.3 Instalar Python Packages
bashpip3 install matplotlib numpy scipy pandas
Por quê? Para análise e visualização de resultados da simulação.

2. Instalação do ns-3
2.1 Baixar ns-3
bash# Criar diretório de trabalho
mkdir -p ~/ns3-workspace
cd ~/ns3-workspace

# Clonar ns-3 (versão de desenvolvimento com WiFi 7)
git clone https://gitlab.com/nsnam/ns-3-dev.git ns-3-dev
cd ns-3-dev
Decisão importante: Usamos ns-3-dev porque o suporte completo ao WiFi 7 (802.11be) está nas versões mais recentes. Versões estáveis antigas podem não ter todos os recursos do WiFi 7.
2.2 Configurar o Build
bash# Configurar com CMake (recomendado para ns-3.36+)
./ns3 configure --enable-examples --enable-tests --build-profile=optimized
Opções explicadas:

--enable-examples: Compila exemplos para referência
--enable-tests: Habilita testes unitários
--build-profile=optimized: Compilação otimizada (mais rápida em execução)

Alternativas: debug (para desenvolvimento), release (máxima otimização)



2.3 Compilar o ns-3
bash# Compilar (use -j$(nproc) para paralelizar)
./ns3 build
Tempo esperado: 10-30 minutos dependendo do hardware.
Se houver erros: Verifique se todas as dependências foram instaladas corretamente.

3. Compilação e Execução
3.1 Criar o Script de Simulação
bash# Criar diretório para nossos scripts
mkdir -p scratch

# Copiar o código da simulação
nano scratch/wifi7-mlo-simulation.cc
Cole o código C++ da simulação WiFi 7 no arquivo e salve (Ctrl+O, Enter, Ctrl+X).
3.2 Compilar o Script
bash# Recompilar o ns-3 incluindo nosso novo script
./ns3 build
Por quê recompilar? O ns-3 precisa indexar e compilar nosso novo arquivo fonte.
3.3 Executar a Simulação
Execução Básica
bash./ns3 run wifi7-mlo-simulation
Execução com Parâmetros Personalizados
bash# Exemplo 1: 10 estações, canal de 320 MHz, MLO habilitado
./ns3 run "wifi7-mlo-simulation --nStations=10 --channelWidth=320 --enableMLO=true"

# Exemplo 2: Alta carga, análise de latência
./ns3 run "wifi7-mlo-simulation --nStations=20 --dataRate=500Mbps --simulationTime=20"

# Exemplo 3: Diferentes distâncias (estudo de coexistência)
./ns3 run "wifi7-mlo-simulation --distance=5 --nStations=15"

# Exemplo 4: Teste de diferentes MCS
./ns3 run "wifi7-mlo-simulation --mcs=13 --channelWidth=320"
Parâmetros disponíveis:

--nStations: Número de dispositivos (1-50+)
--simulationTime: Duração em segundos (1-100+)
--channelWidth: 20, 40, 80, 160, 320 MHz
--mcs: Índice 0-13 (13 = 4096-QAM)
--dataRate: Taxa de tráfego (ex: 50Mbps, 1Gbps)
--enableMLO: true/false
--distance: Distância em metros (1-100)

3.4 Execução com Logging Detalhado
bash# Ver logs detalhados do WiFi
NS_LOG=WifiPhy:WifiMac ./ns3 run wifi7-mlo-simulation

# Apenas warnings e erros
NS_LOG=*=error:*=warn ./ns3 run wifi7-mlo-simulation

4. Análise dos Resultados
4.1 Saída no Console
A simulação imprime diretamente:

Throughput total e por fluxo (Mbps)
Latência média (ms)
Jitter médio (ms)
Taxa de perda de pacotes (%)
Análise de MLO (se habilitado)
Impacto da largura de canal

4.2 Arquivo XML (FlowMonitor)
Gerado automaticamente: wifi7-mlo-flowmon.xml
bash# Ver estatísticas formatadas
./ns3 run "wifi7-mlo-simulation" > resultados.txt
cat resultados.txt
4.3 Análise com Python
Crie um script Python para visualização:
bashnano analyze_results.py
python#!/usr/bin/env python3
import xml.etree.ElementTree as ET
import matplotlib.pyplot as plt
import numpy as np

# Parse do arquivo XML
tree = ET.parse('wifi7-mlo-flowmon.xml')
root = tree.getroot()

throughputs = []
latencies = []
jitters = []

for flow in root.findall('.//Flow'):
    # Extrair métricas
    rx_bytes = int(flow.get('rxBytes', 0))
    tx_packets = int(flow.get('txPackets', 0))
    rx_packets = int(flow.get('rxPackets', 0))
    
    if rx_packets > 0:
        delay_sum = float(flow.get('delaySum', '0').replace('ns', ''))
        jitter_sum = float(flow.get('jitterSum', '0').replace('ns', ''))
        
        # Calcular throughput (Mbps)
        time_first_tx = float(flow.get('timeFirstTxPacket', '0').replace('ns', ''))
        time_last_rx = float(flow.get('timeLastRxPacket', '0').replace('ns', ''))
        duration = (time_last_rx - time_first_tx) / 1e9
        
        if duration > 0:
            throughput = (rx_bytes * 8) / (duration * 1e6)
            throughputs.append(throughput)
        
        # Calcular latência e jitter (ms)
        latency = (delay_sum / rx_packets) / 1e6
        latencies.append(latency)
        
        if rx_packets > 1:
            jitter = (jitter_sum / (rx_packets - 1)) / 1e6
            jitters.append(jitter)

# Criar visualizações
fig, axes = plt.subplots(2, 2, figsize=(12, 10))

# Gráfico 1: Throughput por Fluxo
axes[0, 0].bar(range(len(throughputs)), throughputs, color='steelblue')
axes[0, 0].set_xlabel('Fluxo')
axes[0, 0].set_ylabel('Throughput (Mbps)')
axes[0, 0].set_title('Throughput por Fluxo')
axes[0, 0].grid(True, alpha=0.3)

# Gráfico 2: Latência por Fluxo
axes[0, 1].bar(range(len(latencies)), latencies, color='coral')
axes[0, 1].set_xlabel('Fluxo')
axes[0, 1].set_ylabel('Latência (ms)')
axes[0, 1].set_title('Latência Média por Fluxo')
axes[0, 1].grid(True, alpha=0.3)

# Gráfico 3: Distribuição de Throughput
axes[1, 0].hist(throughputs, bins=10, color='green', alpha=0.7, edgecolor='black')
axes[1, 0].set_xlabel('Throughput (Mbps)')
axes[1, 0].set_ylabel('Frequência')
axes[1, 0].set_title('Distribuição de Throughput')
axes[1, 0].grid(True, alpha=0.3)

# Gráfico 4: Jitter por Fluxo
axes[1, 1].bar(range(len(jitters)), jitters, color='purple')
axes[1, 1].set_xlabel('Fluxo')
axes[1, 1].set_ylabel('Jitter (ms)')
axes[1, 1].set_title('Jitter Médio por Fluxo')
axes[1, 1].grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('wifi7_analysis.png', dpi=300)
print("Gráficos salvos em: wifi7_analysis.png")

# Estatísticas resumidas
print("\\n=== Estatísticas Resumidas ===")
print(f"Throughput Médio: {np.mean(throughputs):.2f} Mbps")
print(f"Throughput Mínimo: {np.min(throughputs):.2f} Mbps")
print(f"Throughput Máximo: {np.max(throughputs):.2f} Mbps")
print(f"Desvio Padrão: {np.std(throughputs):.2f} Mbps")
print(f"\\nLatência Média: {np.mean(latencies):.2f} ms")
print(f"Latência Mínima: {np.min(latencies):.2f} ms")
print(f"Latência Máxima: {np.max(latencies):.2f} ms")
if jitters:
    print(f"\\nJitter Médio: {np.mean(jitters):.2f} ms")
Execute o script:
bashchmod +x analyze_results.py
python3 analyze_results.py

5. Experimentos Sugeridos
5.1 Estudo de Latência sob Carga Crescente
Objetivo: Avaliar como a latência aumenta com a carga de rede.
bash#!/bin/bash
# Script para automatizar múltiplas simulações

echo "Iniciando estudo de latência sob carga..."

# Testar diferentes cargas
for dataRate in 10Mbps 50Mbps 100Mbps 200Mbps 500Mbps 1Gbps; do
    echo "Testando com taxa: $dataRate"
    ./ns3 run "wifi7-mlo-simulation --dataRate=$dataRate --nStations=10 --simulationTime=15" > "results_${dataRate}.txt"
done

echo "Simulações concluídas. Resultados em results_*.txt"
Análise esperada:

Latência baixa em cargas leves (< 100 Mbps)
Aumento exponencial próximo à saturação
WiFi 7 com MLO deve mostrar melhor comportamento sob carga

5.2 Comparação de Larguras de Canal
Objetivo: Comparar desempenho com diferentes larguras de canal.
bash#!/bin/bash
echo "Comparando larguras de canal..."

for width in 20 40 80 160 320; do
    echo "Testando largura: $width MHz"
    ./ns3 run "wifi7-mlo-simulation --channelWidth=$width --nStations=5 --simulationTime=10" > "results_${width}MHz.txt"
done
Análise esperada:

20 MHz: Throughput baixo, mas robusto
40-80 MHz: Bom equilíbrio
160 MHz: Alto throughput, requer bom SNR
320 MHz: Máximo throughput (apenas WiFi 7), muito sensível

5.3 Avaliação de MLO vs. Single-Link
Objetivo: Demonstrar benefícios do MLO.
bash#!/bin/bash
echo "Comparando MLO vs Single-Link..."

# Com MLO
echo "Executando com MLO habilitado..."
./ns3 run "wifi7-mlo-simulation --enableMLO=true --nStations=10 --dataRate=500Mbps" > results_mlo_enabled.txt

# Sem MLO
echo "Executando sem MLO..."
./ns3 run "wifi7-mlo-simulation --enableMLO=false --nStations=10 --dataRate=500Mbps" > results_mlo_disabled.txt

echo "Comparação concluída!"
Benefícios esperados do MLO:

Throughput agregado maior
Latência reduzida via balanceamento de carga
Maior confiabilidade por redundância

5.4 Estudo de Coexistência
Objetivo: Avaliar interferência com múltiplas redes próximas.
bash#!/bin/bash
echo "Estudo de coexistência - densidade de rede..."

# Variar número de estações (densidade)
for stations in 5 10 15 20 25 30; do
    echo "Testando com $stations estações..."
    ./ns3 run "wifi7-mlo-simulation --nStations=$stations --distance=10 --simulationTime=15" > "results_${stations}sta.txt"
done
Análise esperada:

Degradação de desempenho com mais estações
MLO deve mitigar melhor a contenção
Maior largura de canal = mais sensível à densidade

5.5 Teste de MCS Adaptativo
Objetivo: Avaliar impacto de diferentes índices MCS.
bash#!/bin/bash
echo "Testando diferentes valores de MCS..."

# MCS 0-13 (WiFi 7)
for mcs in 0 3 6 9 11 13; do
    echo "Testando MCS $mcs..."
    ./ns3 run "wifi7-mlo-simulation --mcs=$mcs --channelWidth=160 --nStations=5" > "results_mcs${mcs}.txt"
done
Interpretação dos MCS:

MCS 0-2: BPSK/QPSK - Robusto, baixo throughput
MCS 3-4: 16-QAM - Equilíbrio
MCS 5-7: 64-QAM - Alto throughput
MCS 8-9: 256-QAM - Muito alto throughput
MCS 10-11: 1024-QAM - Requer excelente SNR
MCS 12-13: 4096-QAM (WiFi 7) - Máximo throughput, extremamente sensível


6. Interpretação dos Resultados
6.1 Métricas Chave
Throughput

Bom: > 80% da taxa teórica
Aceitável: 50-80%
Problema: < 50% (investigar interferência, MCS, largura de canal)

Latência

Excelente: < 5 ms
Boa: 5-20 ms
Aceitável: 20-50 ms
Alta: > 50 ms (possível congestionamento)

Jitter

Baixo: < 10 ms (ideal para VoIP, gaming)
Moderado: 10-30 ms
Alto: > 30 ms (problemas para aplicações tempo-real)

Taxa de Perda

Ideal: < 0.1%
Aceitável: 0.1-1%
Problema: > 1%

6.2 Fatores que Afetam Desempenho
Largura de Canal
Largura    |  Throughput  |  Alcance  |  Sensibilidade
------------------------------------------------------
20 MHz     |  Baixo       |  Alto     |  Baixa
40 MHz     |  Médio       |  Médio    |  Média
80 MHz     |  Alto        |  Médio    |  Média-Alta
160 MHz    |  Muito Alto  |  Baixo    |  Alta
320 MHz    |  Máximo      |  Mínimo   |  Muito Alta
MCS vs. SNR Requerido
MCS    |  Modulação  |  SNR Mínimo  |  Throughput Relativo
------------------------------------------------------------
0-2    |  BPSK/QPSK  |  5-10 dB     |  Baixo
3-4    |  16-QAM     |  10-15 dB    |  Médio
5-7    |  64-QAM     |  15-20 dB    |  Alto
8-9    |  256-QAM    |  20-25 dB    |  Muito Alto
10-11  |  1024-QAM   |  30-35 dB    |  Extremo
12-13  |  4096-QAM   |  40+ dB      |  Máximo (WiFi 7)
6.3 Troubleshooting Comum
Problema: Throughput muito baixo

Verificar MCS (pode estar muito conservador)
Aumentar largura de canal
Reduzir distância ou adicionar potência
Habilitar MLO

Problema: Alta latência

Reduzir carga de tráfego
Verificar contenção (muitas estações)
Usar MLO para balanceamento
Verificar configuração de buffering

Problema: Taxa de perda alta

Melhorar SNR (reduzir distância/interferência)
Reduzir MCS
Diminuir largura de canal
Verificar modelo de propagação


7. Recursos Adicionais
7.1 Documentação

ns-3 Official Docs: https://www.nsnam.org/documentation/
WiFi Module: https://www.nsnam.org/docs/models/html/wifi.html
Tutorial ns-3: https://www.nsnam.org/docs/tutorial/html/

7.2 WiFi 7 (802.11be) Especificações

IEEE 802.11be: Standard oficial
Recursos principais:

MLO (Multi-Link Operation)
320 MHz channel width (6 GHz)
4096-QAM (MCS 12-13)
Multi-RU (Resource Unit)
Enhanced MU-MIMO



7.3 Scripts de Automação Úteis
Comparação Completa
bash#!/bin/bash
# comparacao_completa.sh - Executa todos os experimentos

echo "=== Bateria Completa de Testes WiFi 7 ==="

# Criar diretório de resultados
mkdir -p results_$(date +%Y%m%d_%H%M%S)
cd results_$(date +%Y%m%d_%H%M%S)

# 1. Larguras de canal
echo "[1/4] Testando larguras de canal..."
for width in 20 40 80 160 320; do
    ../ns3 run "wifi7-mlo-simulation --channelWidth=$width" > channel_${width}.txt
done

# 2. MLO comparison
echo "[2/4] Comparando MLO..."
../ns3 run "wifi7-mlo-simulation --enableMLO=true" > mlo_enabled.txt
../ns3 run "wifi7-mlo-simulation --enableMLO=false" > mlo_disabled.txt

# 3. Escalabilidade
echo "[3/4] Testando escalabilidade..."
for sta in 5 10 15 20 25; do
    ../ns3 run "wifi7-mlo-simulation --nStations=$sta" > scalability_${sta}.txt
done

# 4. Carga crescente
echo "[4/4] Testando sob carga..."
for rate in 50Mbps 100Mbps 200Mbps 500Mbps 1Gbps; do
    ../ns3 run "wifi7-mlo-simulation --dataRate=$rate" > load_${rate}.txt
done

echo "Testes concluídos! Resultados em: $(pwd)"

8. Conclusões e Próximos Passos
8.1 O que foi Aprendido

✅ Configuração completa do ambiente ns-3 no WSL
✅ Implementação de simulação WiFi 7 com MLO
✅ Análise de métricas (throughput, latência, jitter)
✅ Experimentos comparativos de desempenho

8.2 Extensões Possíveis

Mobilidade: Adicionar padrões de movimento realistas
Tráfego misto: Combinar TCP + UDP, diferentes QoS
Interferência: Simular múltiplas redes coexistentes
Energy models: Avaliar consumo de energia
Machine Learning: Rate adaptation inteligente

8.3 Publicação de Resultados
Para trabalhos acadêmicos, inclua:

Parâmetros de simulação detalhados
Modelo de propagação usado
Configurações de PHY/MAC
Múltiplas execuções (variação estatística)
Intervalos de confiança (95%)


9. Referências Técnicas
Decisões de Design Explicadas
Por que ns-3?

Precisão: Simulador de eventos discretos de alta fidelidade
Validado: Amplamente usado em pesquisa acadêmica
WiFi 7: Suporte aos recursos mais recentes do 802.11be
Open-source: Código auditável e extensível

Por que UDP em vez de TCP?

Latência pura: TCP adiciona retransmissões e controle de congestionamento
Análise limpa: Evita complexidade do TCP para métricas base
Tempo-real: Muitas aplicações WiFi 7 (AR/VR) usam UDP

Por que FlowMonitor?

Estatísticas detalhadas: Por fluxo individual
Não-intrusivo: Não afeta simulação
Exportável: XML para processamento posterior
