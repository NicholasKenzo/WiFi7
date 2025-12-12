💾 Guia Completo: Simulação Wi-Fi 7 (802.11be) no ns-3 (WSL/Ubuntu)Um guia passo a passo para configurar, executar e analisar simulações do padrão IEEE 802.11be (Wi-Fi 7) utilizando o Network Simulator 3 (ns-3) em um ambiente WSL Ubuntu.📖 SumárioPreparação do Ambiente WSLInstalação e Configuração do ns-3Compilação e Execução da SimulaçãoAnálise dos Resultados (Python & FlowMonitor)Experimentos SugeridosInterpretação e TroubleshootingRecursos Adicionais e Referências1. Preparação do Ambiente WSL1.1 Atualizar o SistemaGarantir que as bibliotecas e ferramentas de build estejam nas versões mais recentes.Bash# Atualizar repositórios e pacotes
sudo apt update && sudo apt upgrade -y
1.2 Instalar Dependências do ns-3Instalação de compiladores, bibliotecas de parsing, binding Python, e ferramentas de visualização (como GTK/Qt para o NetAnim e MPI para simulações paralelas).Bash# Compiladores e ferramentas de build
sudo apt install -y build-essential gcc g++ cmake ninja-build

# Python e ferramentas relacionadas
sudo apt install -y python3 python3-pip python3-dev

# Bibliotecas necessárias para ns-3 e ferramentas de suporte
sudo apt install -y git mercurial wget libsqlite3-dev libxml2 libxml2-dev
sudo apt install -y libgtk-3-dev qtbase5-dev qtchooser qt5-qmake qtbase5-dev-tools
sudo apt install -y gir1.2-goocanvas-2.0 python3-gi python3-gi-cairo python3-pygraphviz gir1.2-gtk-3.0 ipython3
sudo apt install -y openmpi-bin openmpi-common openmpi-doc libopenmpi-dev
sudo apt install -y gdb valgrind
sudo apt install -y doxygen graphviz imagemagick
sudo apt install -y texlive texlive-extra-utils texlive-latex-extra texlive-font-utils dvipng latexmk
1.3 Instalar Python Packages para AnálisePacotes cruciais para a análise e visualização de dados (gráficos, estatísticas).Bashpip3 install matplotlib numpy scipy pandas
2. Instalação e Configuração do ns-3Utilizamos a versão de desenvolvimento (ns-3-dev) pois o suporte completo e mais recente ao padrão Wi-Fi 7 (802.11be/EHT) está em constante evolução nesta branch.2.1 Baixar ns-3Bash# Criar diretório de trabalho
mkdir -p ~/ns3-workspace
cd ~/ns3-workspace

# Clonar ns-3 (versão de desenvolvimento com WiFi 7)
git clone https://gitlab.com/nsnam/ns-3-dev.git ns-3-dev
cd ns-3-dev
2.2 Configurar o BuildConfiguração utilizando CMake e otimizando o build para execução rápida.Bash# Configurar com CMake (recomendado para ns-3.36+)
./ns3 configure --enable-examples --enable-tests --build-profile=optimized
OpçãoPropósito--enable-examplesCompila scripts de exemplo do ns-3.--enable-testsHabilita testes unitários (boa prática).--build-profile=optimizedCompilação otimizada para performance (default para execução).2.3 Compilar o ns-3Bash# Compilar (use -j$(nproc) para paralelizar o processo com todos os núcleos disponíveis)
./ns3 build -j$(nproc)
Tempo Esperado: 10-30 minutos, dependendo do hardware do WSL.3. Compilação e Execução da Simulação3.1 Estrutura do Script de SimulaçãoAssumimos que o código C++ da sua simulação Wi-Fi 7 (ex: wifi7-mlo-simulation.cc) será colocado na pasta scratch/.Bash# Criar diretório para nossos scripts
mkdir -p scratch

# Copiar ou criar o código da simulação (Exemplo: utilizando nano)
nano scratch/wifi7-mlo-simulation.cc
# Cole o código C++ aqui e salve.
3.2 Compilar o ScriptÉ necessário re-compilar o ns-3 sempre que um novo arquivo C++ é adicionado ou modificado na pasta scratch/.Bash./ns3 build
3.3 Executar a SimulaçãoExecução BásicaBash./ns3 run wifi7-mlo-simulation
Execução com Parâmetros PersonalizadosO script permite variar parâmetros cruciais para os experimentos de Wi-Fi 7.Bash# Exemplo 1: 10 estações, canal de 320 MHz, MLO habilitado.
./ns3 run "wifi7-mlo-simulation --nStations=10 --channelWidth=320 --enableMLO=true"

# Exemplo 2: Teste de diferentes MCS (4096-QAM)
./ns3 run "wifi7-mlo-simulation --mcs=13 --channelWidth=320"
ParâmetroDescriçãoValores Comuns--nStationsNúmero de dispositivos conectados.1, 10, 20--simulationTimeDuração da simulação em segundos.10, 20, 100--channelWidthLargura de banda do canal em MHz.20, 80, 160, 320--mcsÍndice de Modulação e Codificação.0 - 13 (MCS 13 = 4096-QAM)--enableMLOAtiva a Operação Multi-Link (Wi-Fi 7).true / false4. Análise dos Resultados (Python & FlowMonitor)O ns-3 utiliza o FlowMonitor para exportar estatísticas detalhadas de throughput, delay e jitter por fluxo em formato XML.4.1 Exportar EstatísticasA saída do console e o arquivo XML (ex: wifi7-mlo-flowmon.xml) são gerados automaticamente na raiz do ns-3-dev/.Bash# Redirecionar a saída do console para um arquivo de texto
./ns3 run "wifi7-mlo-simulation" > resultados.txt
4.2 Análise com PythonO script Python abaixo processa o XML do FlowMonitor, calcula métricas e gera visualizações (wifi7_analysis.png).Bashnano analyze_results.py
Python#!/usr/bin/env python3
import xml.etree.ElementTree as ET
import matplotlib.pyplot as plt
import numpy as np

# Parse do arquivo XML (Certifique-se que o nome do arquivo corresponde ao gerado)
tree = ET.parse('wifi7-mlo-flowmon.xml') 
root = tree.getroot()

throughputs = []
latencies = []
jitters = []

# (O código de processamento de dados e plotagem continua aqui conforme o seu exemplo)

# ...

plt.tight_layout()
plt.savefig('wifi7_analysis.png', dpi=300)
print("Gráficos salvos em: wifi7_analysis.png")
print(f"\nThroughput Médio: {np.mean(throughputs):.2f} Mbps")
print(f"Latência Média: {np.mean(latencies):.2f} ms")
Execute o script:Bashchmod +x analyze_results.py
python3 analyze_results.py
5. Experimentos SugeridosUtilize scripts shell simples para automatizar a execução de múltiplas simulações.5.1 Estudo de Latência sob Carga CrescenteAvaliar a saturação da rede variando a taxa de tráfego (--dataRate).Bash#!/bin/bash
echo "Iniciando estudo de latência sob carga crescente..."
for dataRate in 10Mbps 50Mbps 100Mbps 200Mbps 500Mbps 1Gbps; do
    echo "Testando com taxa: $dataRate"
    ./ns3 run "wifi7-mlo-simulation --dataRate=$dataRate --nStations=10 --simulationTime=15" > "results_${dataRate}.txt"
done
5.2 Comparação de Larguras de CanalDemonstrar o ganho de throughput e a sensibilidade do sistema ao variar a largura de canal.Bash#!/bin/bash
echo "Comparando larguras de canal (20 MHz a 320 MHz)..."
for width in 20 40 80 160 320; do
    echo "Testando largura: $width MHz"
    ./ns3 run "wifi7-mlo-simulation --channelWidth=$width --nStations=5 --simulationTime=10" > "results_${width}MHz.txt"
done
5.3 Avaliação de MLO vs. Single-LinkIsolar o benefício do MLO em throughput agregado e latência.Bash#!/bin/bash
echo "Comparando MLO vs Single-Link..."

# Com MLO (Multi-Link)
./ns3 run "wifi7-mlo-simulation --enableMLO=true --nStations=10 --dataRate=500Mbps" > results_mlo_enabled.txt

# Sem MLO (Single-Link)
./ns3 run "wifi7-mlo-simulation --enableMLO=false --nStations=10 --dataRate=500Mbps" > results_mlo_disabled.txt
6. Interpretação e Troubleshooting6.1 Fatores que Afetam o DesempenhoLarguraThroughput RelativoAlcanceSensibilidade (SNR)20 MHzBaixoAltoBaixa80 MHzAltoMédioMédia-Alta320 MHzMáximoMínimoMuito AltaMCSModulaçãoSNR MínimoThroughput0-2BPSK/QPSK5-10 dBBaixo, Robusto10-111024-QAM30-35 dBExtremo12-134096-QAM40+ dBMáximo, Muito Sensível6.2 Troubleshooting ComumProblemaPossível CausaAções SugeridasThroughput baixoMCS muito baixo ou canal estreito.Aumentar --mcs ou --channelWidth.Latência AltaCongestionamento (--dataRate muito alto).Reduzir carga, diminuir nStations ou habilitar MLO.Taxa de Perda AltaSinal Fraco ou Ruído (4096-QAM).Reduzir --mcs (ex: de 13 para 11) ou diminuir --channelWidth.7. Recursos Adicionais e Referênciasns-3 Documentação Oficial:Tutorial ns-3Documentação do Módulo Wi-FiEspecificações do IEEE 802.11be (Wi-Fi 7):MLO (Multi-Link Operation)320 MHz channel width (Banda de 6 GHz)4096-QAM (MCS 12-13)⚖️ LicençaEste projeto e os scripts de exemplo estão distribuídos sob a Licença MIT.Gostaria de ajuda para criar o script C++ (wifi7-mlo-simulation.cc) para um dos cenários sugeridos, como a comparação MLO vs. Single-Link?
