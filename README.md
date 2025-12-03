# Keyboard-Menu---workspace

![Visitantes do Projeto](https://visitor-badge.laobi.icu/badge?page_id=arvoredossaberes.keyboard-menu---workspace)
[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](LICENSE)
![C++](https://img.shields.io/badge/C%2B%2B-17-blue)
![CMake](https://img.shields.io/badge/CMake-%3E%3D3.16-informational)
[![Docs](https://img.shields.io/badge/docs-Doxygen-blueviolet)](docs/index.html)
[![Latest Release](https://img.shields.io/github/v/release/ArvoreDosSaberes/keyboard-menu---workspace?label=version)](https://github.com/ArvoreDosSaberes/keyboard-menu---workspace/releases/latest)
[![Contributions Welcome](https://img.shields.io/badge/contributions-welcome-success.svg)](#contribuindo)

Workspace demonstrando como usar as bibliotecas `Keyboard_Menu_FreeRTOS` e `Keyboard` em projetos C++/CMake, além de integrar recursos de **display OLED SSD1306** (`oled_ssd1306`) e **log colorido em terminal VT100** (`log_vt100`).

## Como clonar o repositório com submódulos

Este workspace depende de várias bibliotecas adicionadas como **submódulos Git** (`keyboard`, `keyboard_menu_freertos`, `oled`, `log_vt100`, `i2c-proxy`, `FreeRTOS-Kernel`, etc.).

Para clonar já trazendo todos os submódulos:

```bash
git clone --recurse-submodules git@github.com:ArvoreDosSaberes/Keyboard-Menu---workspace.git
cd Keyboard-Menu---workspace
```

Se você já clonou **sem** `--recurse-submodules`, inicialize e atualize depois com:

```bash
git submodule update --init --recursive
```

E para atualizar os submódulos para as referências registradas no commit atual do workspace:

```bash
git submodule update --recursive
```

Se quiser buscar as últimas revisões remotas de cada submódulo (por exemplo durante o desenvolvimento):

```bash
git submodule update --remote --merge --recursive
```

## Visão geral do projeto

Este repositório funciona como **workspace de exemplo** para organizar e demonstrar o uso de duas bibliotecas relacionadas a leitura de teclado matricial e navegação por menus, bem como bibliotecas auxiliares de display, log e acesso a barramento I2C:

- **`Keyboard`**: biblioteca de leitura e tratamento de teclado (por exemplo, teclado matricial ou conjunto de botões), focada na **detecção de teclas**, debounce e mapeamento lógico.
- **`Keyboard_Menu_FreeRTOS`**: biblioteca que utiliza um teclado (via `Keyboard`) para navegação em **menus hierárquicos**, pensada para ambientes com **FreeRTOS** (tarefas, filas, etc.).
- **`oled_ssd1306`**: biblioteca para controle de display OLED SSD1306 via I2C (API de alto nível `oled.h/.c`).
- **`log_vt100`**: biblioteca de log colorido em terminal VT100/ANSI, com níveis de log e suporte a formato binário (`%b`).
 - **`i2c-proxy` (`I2C_Proxy`)**: biblioteca C++ que encapsula o periférico I2C do RP2040/RP2350, com suporte opcional a FreeRTOS via semáforos.

O objetivo deste workspace é servir como **referência de integração** das bibliotecas em um projeto real, facilitando o reuso em outros firmwares ou aplicações embarcadas.

## Estrutura básica do workspace

De forma geral, você encontrará algo semelhante a:

- `lib/Keyboard` – código-fonte da biblioteca de teclado.
- `lib/Keyboard_Menu_FreeRTOS` – código-fonte da biblioteca de menu baseada em teclado e FreeRTOS.
- `lib/oled` – código-fonte da biblioteca OLED SSD1306 (`oled_ssd1306`).
- `lib/log_vt100` – submódulo com a biblioteca de log VT100/ANSI para depuração.
 - `lib/i2c-proxy` – submódulo com a biblioteca `I2C_Proxy`, wrapper para o periférico I2C e integração com FreeRTOS.
- `examples/` ou `src/` – exemplos e/ou aplicações de demonstração (dependendo de como o projeto estiver organizado).
- `CMakeLists.txt` – configuração principal do CMake para montar o workspace e vincular as bibliotecas.

> A estrutura exata pode variar conforme evolução do projeto, mas a ideia central é manter as bibliotecas isoladas em `lib/` e exemplos/demonstrações em outro diretório.

## Usando a biblioteca `Keyboard` isoladamente

Esta biblioteca é indicada quando você precisa **apenas ler o teclado** e tratar eventos de tecla em seu código.

### Passos gerais de uso

1. **Adicionar a biblioteca ao seu projeto**
   - Inclua o diretório `lib/Keyboard` no seu CMake (por exemplo, usando `add_subdirectory(lib/Keyboard)` ou similar).
   - Em seguida, ligue a biblioteca alvo ao seu executável (por exemplo, `target_link_libraries(seu_alvo PRIVATE Keyboard)` – o nome real pode variar conforme o CMake desta lib).

2. **Configurar o hardware de teclado**
   - Inicialize os pinos (linhas e colunas) do seu teclado matricial ou entradas digitais.
   - Ajuste o mapeamento de teclas (por exemplo, matriz de caracteres ou códigos).

3. **Ler teclas no loop principal ou em tarefa dedicada**
   - Chame periodicamente as funções de varredura da biblioteca.
   - Trate eventos como **tecla pressionada**, **mantida** ou **solta**, conforme a API oferecida.

4. **Integrar com a lógica da aplicação**
   - A partir dos eventos de tecla, chame suas rotinas de negócio (navegação de telas, alteração de parâmetros, etc.).

## Usando a biblioteca `Keyboard_Menu_FreeRTOS` isoladamente

Esta biblioteca é pensada para cenários em que você já possui **FreeRTOS** disponível e quer criar um **menu navegável via teclado**.

### Passos gerais de uso

1. **Adicionar a biblioteca ao projeto com FreeRTOS**
   - Inclua `lib/Keyboard_Menu_FreeRTOS` via CMake.
   - Garanta que FreeRTOS já esteja configurado e incluído no seu ambiente.

2. **Definir a estrutura de menu**
   - Crie itens de menu (por exemplo, nós representando telas, opções e ações).
   - Defina callbacks ou handlers a serem executados quando uma opção é selecionada.

3. **Configurar a tarefa de menu**
   - Crie uma **task do FreeRTOS** responsável por:
     - Ler eventos de tecla (diretamente ou via fila/queue compartilhada).
     - Atualizar o estado do menu.
     - Disparar ações associadas às opções.

4. **Atualizar a interface (display/terminal)**
   - Dentro da task ou em outra unidade de código, atualize o display, LCD ou saída serial conforme o item de menu atual.

## Usando `Keyboard` e `Keyboard_Menu_FreeRTOS` em conjunto

O uso combinado segue a seguinte ideia:

1. **`Keyboard` cuida do hardware**
   - Varredura de linhas/colunas.
   - Debounce.
   - Conversão de combinações em códigos de tecla lógicos (por exemplo: UP, DOWN, LEFT, RIGHT, ENTER, BACK).

2. **`Keyboard_Menu_FreeRTOS` cuida da navegação e lógica de menu**
   - Recebe os eventos de tecla (por exemplo via fila do FreeRTOS ou função de callback).
   - Atualiza o estado do menu, selecionando itens e executando ações.

### Fluxo típico de integração

1. **Inicialização**
   - Configure o teclado com a biblioteca `Keyboard`.
   - Crie as tasks do FreeRTOS necessárias (por exemplo, uma para leitura de teclado e outra para navegação de menu).

2. **Comunicação entre tarefas**
   - A task de teclado envia eventos de tecla para uma **queue**.
   - A task de menu (`Keyboard_Menu_FreeRTOS`) consome esses eventos e ajusta o menu.

3. **Atualização visual**
   - Após processar uma tecla, o menu notifica ou chama funções responsáveis por atualizar o display/saída.

## Como compilar e executar os exemplos

Os comandos exatos podem variar, mas um fluxo típico com CMake é:

```bash
mkdir -p build
cd build
cmake ..
cmake --build .
```

Caso o projeto contenha **exemplos específicos**, eles poderão ser gerados como executáveis dentro de `build/`. Consulte também a documentação gerada pelo **Doxygen** (link no badge de docs) para detalhes de API.

## Contribuindo

Contribuições são bem-vindas! Sugestões de melhoria incluem:

- **Novos exemplos** de uso das bibliotecas em diferentes microcontroladores ou ambientes.
- **Aprimoramento da documentação** (incluindo esquemas de ligação de hardware e diagramas de menu).
- Correções de bugs e otimizações de desempenho.

Antes de abrir um *pull request*, procure:

- Seguir o padrão de código existente.
- Manter os nomes em inglês para classes, funções e variáveis.
- Adicionar comentários e/ou documentação Doxygen quando alterar APIs públicas.

