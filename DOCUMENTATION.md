# Documentação do VoxBrasil

## 1. Visão Geral

VoxBrasil é um sistema VOIP para SA:MP / open.mp com duas partes principais:
- `client/`: código do cliente que roda no GTA San Andreas (plugin `.asi`)
- `server/`: código do servidor que roda como plugin no servidor SA:MP/open.mp

A arquitetura é cliente-servidor:
- Cliente captura e reproduz áudio localmente
- Servidor gerencia streams, ouvintes e falantes, e repassa a voz entre jogadores

## 2. Suporte e propósito

Base de suporte original:
- Cliente: SA:MP 0.3.7 (R1, R3)
- Servidor: open.mp (última versão da época do código)

O projeto expõe uma API Pawn (`sampvoice.inc`) para scripts do servidor controlarem streams de voz, ativação de microfone, efeitos, e gerenciamento de ouvintes/falantes. O include permanece `sampvoice.inc` nesta versão 1.0.0 para compatibilidade.

## 3. Configuração e build

### Requisitos básicos

- `git`
- `cmake`
- `Visual Studio` com suporte Win32
- `DirectX SDK` (para compilar o cliente)
- `bass`, `bass_fx` e `opus` já estão referenciadas no link do CMake

### Passos de build

1. Clone o projeto:
```powershell
git clone https://github.com/AmyrAhmady/sampvoice.git
```
2. Atualize submódulos:
```powershell
git submodule update --init --recursive
```
3. Crie a pasta de build:
```powershell
mkdir build
cd build
```
4. Gere os arquivos de build:
```powershell
cmake .. -A Win32
```
5. Abra o `.sln` gerado no `build` e compile.

### Observações

- O CMake espera a variável de ambiente `DXSDK_DIR` apontando para o DirectX SDK.
- O cliente é gerado como `voxbrasil.asi`
- O servidor é gerado como `voxbrasil.dll` em Windows e `voxbrasil.so` em Linux

## 4. Estrutura de pastas

### Raiz do projeto

- `CMakeLists.txt`: build root. Adiciona subdiretórios `lib/omp-sdk`, `lib/omp-net` e `server`. O cliente só é adicionado se a opção `BUILD_CLIENT` estiver ativada.
- `.gitmodules`: define os submódulos `lib/omp-sdk` e `lib/omp-net` necessários para o servidor.
- `README.md`: instruções gerais de instalação e uso.
- `LICENSE`: licença do projeto.
- `docker/`: scripts para build via Docker.
- `setup/`: provavelmente scripts ou artefatos de instalação.

### Diretório `client/`

Contém o plugin do cliente responsável por: 
- captura do microfone,
- reprodução de áudio,
- gestão de streams de voz no jogo,
- integração com DirectX/BASS.

Arquivos importantes:
- `client/CMakeLists.txt`: build do plugin cliente. Usa `DXSDK_DIR` e linka `bass_fx.lib`, `bass.lib`, `opus.lib`.
- `client/main.cpp`: ponto de entrada do plugin cliente.
- `client/Plugin.cpp`, `Plugin.h`: lógica de inicialização e integração com SA:MP.
- `client/Network.cpp`, `Network.h`: gerenciamento de conexão/packet de voz.
- `client/Stream*.cpp/h`: implementação de diferentes tipos de stream (global, local, at player, at vehicle, etc.).
- `client/Effect*.cpp/h`: implementação de efeitos de áudio (chorus, compressor, reverb, etc.).
- `client/resources/`: recursos usados pelo plugin.

### Diretório `server/`

Contém o plugin do servidor responsável por:
- gerenciar streams,
- armazenar listeners e speakers,
- enviar/receber eventos do cliente,
- fornecer a interface Pawn.

Arquivos importantes:
- `server/CMakeLists.txt`: build do plugin servidor.
- `server/main.cpp`: inicialização do plugin servidor.
- `server/NetHandler.cpp/h`: gerencia rede entre cliente e servidor.
- `server/Stream*.cpp/h`: implementação das streams do servidor (global, local, dinâmicas e estáticas em objetos/jogadores/veículos/pontos).
- `server/PlayerStore.cpp/h`: controle de jogadores conectados e estado dos streams.
- `server/Pawn.cpp/h`: registro das natives Pawn e callbacks.
- `server/sampvoice.inc`: arquivo de include Pawn que define a API disponível para scripts.

### Diretórios de biblioteca

- `lib/omp-sdk`: SDK do open.mp usado pelo plugin servidor.
- `lib/omp-net`: biblioteca de rede usada pelo servidor.

Esses dois diretórios são submódulos Git e devem ser inicializados com `git submodule update --init --recursive`.

## 5. O que cada arquivo e pasta trata

### Arquivos principais da raiz
- `README.md`: documentação básica e tutoriais de instalação.
- `CMakeLists.txt`: orquestra build de servidor e opcionalmente do cliente.
- `.gitmodules`: registra submódulos necessários.

### Cliente (`client/`)
- `main.cpp`: inicializa o plugin de voz no cliente.
- `Plugin.cpp/.h`: lógica de integração com SA:MP e hooks do jogo.
- `Network.cpp/.h`: envia e recebe pacotes de voz para o servidor e de outros jogadores.
- `Record.cpp/.h`: captura áudio do microfone usando BASS/Opus.
- `Playback.cpp/.h`: reproduz áudio recebido.
- `Stream*.cpp/.h`: define streams e comportamentos de audição (local/global/por objeto/jogador/veículo/ponto).
- `Effect*.cpp/.h`: cria e aplica efeitos de áudio ao stream.
- `include/`: cabeçalhos públicos e auxiliares usados na compilação do cliente.
- `libraries/`: bibliotecas predefinidas de terceiros (BASS, Opus, etc.).
- `resources/`: ícones e recursos do plugin.

### Servidor (`server/`)
- `main.cpp`: inicializa o plugin servidor.
- `Pawn.cpp/.h`: declara e registra as natives Pawn disponíveis.
- `NetHandler.cpp/.h`: interpreta e responde a pacotes de rede de clientes.
- `Stream*.cpp/.h`: implementa lógicas de stream no servidor.
- `PlayerStore.cpp/.h`: guarda estados de players e streams.
- `ControlPacket.cpp/.h`: define os pacotes de controle usados entre cliente e servidor.
- `sampvoice.inc`: API Pawn pública para scripts do servidor.

## 6. Como atualizar para funções atuais de 2026

### 6.1 Atualizar dependências

- Atualize `lib/omp-sdk` e `lib/omp-net` para as versões atuais do open.mp.
- Substitua o DirectX SDK por Windows SDK moderno ou por APIs de áudio mais recentes como WASAPI se possível.
- Atualize `opus` e `bass` para versões recentes ou substitua `bass` por solução open-source mais atual.
- Garanta compatibilidade com os compiladores modernos do Visual Studio 2022/2025.

### 6.2 Build moderno

- Atualize `CMakeLists.txt` para usar `target_compile_features()` e `target_link_directories()` modernos.
- Adicione suporte a `x64` e `Win32` explicitamente.
- Use `find_package()` para bibliotecas quando disponíveis, em vez de caminhos hardcoded.
- Considere adicionar `FetchContent` para dependências externas em vez de submódulos fixos.

### 6.3 Arquitetura e compatibilidade

- Se o objetivo for compatibilidade com SA:MP e open.mp atuais, mantenha a API Pawn mas revise:
  - nomes de natives,
  - parâmetros de tipos Pawn,
  - callbacks do servidor e eventos de cliente.
- Para SA:MP 2026, avalie se há novos requisitos de segurança ou multiplataforma.
- Atualize o plugin para suportar `Linux x86` e possivelmente `x64` no servidor.
- Considere separar mais claramente a lógica de codec e a lógica do jogo.

### 6.4 Modernização do cliente de voz

- Avalie remoção da dependência direta do `BASS` e implemente abstração de áudio.
- Adicione suporte para microfones modernos e captura multiplataforma, caso queira ir além do Windows.
- Melhore o gerenciamento de latência e jitter no transporte de voz.
- Atualize o codec de voz Opus para as versões mais recentes com melhores configurações de taxa e qualidade.

### 6.5 Atualização da interface Pawn

- Mantenha `sampvoice.inc` compatível com scripts existentes.
- Se novos recursos forem adicionados em 2026, crie natives adicionais em `server/Pawn.cpp`.
- Documente todos os natives e suas assinaturas em um arquivo de referência.

## 7. Pontos de atenção

- O projeto atual utiliza macros antigas e um estilo de build que depende de variáveis de ambiente (`DXSDK_DIR`). Isso pode dificultar a compilação em máquinas modernas.
- O cliente ainda é construído como `SHARED` com prefixo vazio e sufixo `.asi`.
- O servidor usa `OMP-SDK` e `OMP-Network`, então sempre verifique se os submódulos estão atualizados.

## 8. Recomendações para 2026

- Crie uma versão de documentação atualizada dentro do repositório com exemplos Pawn recentes.
- Implemente CI/CD (GitHub Actions) para builds Windows e Linux.
- Adicione testes de build e verificação de submodule.
- Considere migrar a parte cliente para uma arquitetura de plugin mais moderna ou para um mod loader suportado em 2026.

---

### Conclusão

Este documento descreve a arquitetura do VOIP, os arquivos principais, a configuração de build e como evoluir o projeto para um cenário mais atual. Para atualizar em 2026, o foco deve ser em dependências modernas, suporte a compiladores atuais, x64, e APIs de áudio mais atuais.
