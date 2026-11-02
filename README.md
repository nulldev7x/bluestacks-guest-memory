# 🎮 External RPM/WPM para BlueStacks 5 (example-mem)

[![C++20](https://img.shields.io/badge/Standard-C%2B%2B20-blue.svg)](https://en.wikipedia.org/wiki/C%2B%2B20)
[![Platform](https://img.shields.io/badge/Platform-Windows%20x64-0078D6.svg)](https://www.microsoft.com/windows)
[![Target](https://img.shields.io/badge/Target-BlueStacks%205%20(HD--Player)-brightgreen.svg)](https://www.bluestacks.com/)
[![Developer](https://img.shields.io/badge/Author-nulldev7x-blueviolet.svg)](https://github.com/nulldev7x)

Módulo de exemplo e implementação da minha base de leitura e escrita externa de memória (RPM/WPM) para o emulador BlueStacks 5, desenvolvida e mantida por **nulldev7x**.

---

## 📑 Sumário

- [Visão Geral](#-visão-geral)
- [Escopo do Projeto](#-escopo-do-projeto)
- [Minha Implementação de Memória e Offsets](#-minha-implementação-de-memória-e-offsets)
- [Componentes Desenvolvidos](#-componentes-desenvolvidos)
- [Como Compilar](#-como-compilar)
- [Exemplo de Uso](#-exemplo-de-uso)
- [Solução de Problemas](#-solução-de-problemas)
- [Autor e Contato](#-autor-e-contato)
- [Uso Responsável](#-uso-responsável)

---

## 🔍 Visão Geral

Desenvolvi este módulo para viabilizar o acesso direto à memória guest de aplicações Android emuladas:
- **RPM (*Read Process Memory*):** Leitura de tipos primitivos, estruturas e ponteiros da memória guest a partir do Windows.
- **WPM (*Write Process Memory*):** Modificação e escrita direta de dados na memória do aplicativo emulado.

O mecanismo opera externamente através de Virtual Machine Introspection (VMI), conectando-se ao `HD-Player.exe` e traduzindo endereços virtuais do Android diretamente para a memória do host.

---

## 📌 Escopo do Projeto

> [!NOTE]
> - ✅ **O que implementei aqui:** A camada completa de acesso a baixo nível, resolução de CR3 por varredura de kernel, busca de módulos `.so` em VMAs e um exemplo demonstrativo de consulta via terminal de comando.
> - ❌ **O que não faz parte deste repositório:** Interface gráfica de usuário (UI/ImGui), overlays visuais de renderização (ESP desenhado na tela) ou cheats prontos. Esta é uma base técnica de desenvolvimento.

---

## 🧭 Minha Implementação de Memória e Offsets

1. **Offsets:** Deslocamentos estáticos em relação à base de uma biblioteca (como `libil2cpp.so + 0x9EC1C48`). Eles apenas determinam **onde** os campos de dados estão localizados e precisam ser atualizados caso o aplicativo sofra alterações.
2. **Motor de Memória (Engine):** Código que escrevi para cuidar de **como** a leitura/escrita ocorre. Ele recebe o endereço virtual do Android (**GVA**), resolve as tabelas de páginas do guest (**GPA**) e acessa o endereço mapeado no Windows (**HVA**).

---

## 🧱 Componentes Desenvolvidos

- **`workspace/bluestacks.*`:** Controle de alto nível da sessão do BlueStacks, busca da tarefa Android pelo nome do pacote e resolução de módulos.
- **`workspace/memory_engine.*`:** Camada de baixo nível de comunicação com o processo `HD-Player.exe` e gerenciamento de cache.
- **`workspace/gva_memory_bridge.*`:** Conversão de endereços virtuais do guest para o espaço de memória do host.
- **`workspace/kernel_structs.*`:** Mapeamento das estruturas do kernel Linux (`task_struct`, `mm_struct`, `vm_area_struct`).
- **`workspace/memory.hpp`:** Interface com métodos `read<T>()` e `write<T>()`.
- **`example-mem.cpp`:** Demonstração prática imprimindo valores consultados no console.

---

## 🔨 Como Compilar

1. Abra o arquivo `example-mem.vcxproj` no **Visual Studio 2022**.
2. Defina a configuração como **`Release`** e a plataforma como **`x64`**.
3. Confirme que o padrão de linguagem é **ISO C++20** (`/std:c++20`).
4. Compile a solução (`Ctrl + Shift + B`).
5. Execute o binário resultante **como Administrador** com o BlueStacks já em execução.

---

## 💻 Exemplo de Uso

```cpp
#include <impl/includes.hpp>
#include <iostream>

int main() {
    // Inicializa a engine com o pacote do aplicativo alvo
    if (!bluestacks->setup("com.dts.freefireth")) {
        std::cerr << "[-] Erro ao conectar ao BlueStacks. Execute como Administrador.\n";
        return 1;
    }

    // Busca a base da biblioteca
    u32 libil2cpp = bluestacks->get_module_base("libil2cpp.so", 1);
    if (!libil2cpp) {
        std::cerr << "[-] libil2cpp.so nao encontrada!\n";
        return 1;
    }

    std::cout << "[+] Base libil2cpp: 0x" << std::hex << libil2cpp << "\n";

    // Leitura tipada
    auto base_game_facade = memory->read<u32>(libil2cpp + 0x9EC1C48);
    auto game_facade      = memory->read<u32>(base_game_facade);

    std::cout << "[+] GameFacade: 0x" << std::hex << game_facade << "\n";

    return 0;
}
```

---

## 🛠 Solução de Problemas

- **Falha no `setup()`:** Execute o executável com privilégios elevados de Administrador (UAC) e certifique-se de que o jogo já está carregado no BlueStacks.
- **Base do módulo zerada:** Experimente alterar o índice de mapeamento VMA (`get_module_base("libil2cpp.so", 2)`).
- **Leituras nulas:** Verifique a validade dos offsets para a versão atual do aplicativo.

---

## 👤 Autor e Contato

- **Desenvolvedor:** **nulldev7x**
- **Discord:** `@nulldev7x`
- **GitHub:** [github.com/nulldev7x](https://github.com/nulldev7x)

---

## ⚖️ Uso Responsável

Este código foi criado com finalidade puramente **educacional e de estudo de engenharia reversa e arquitetura de sistemas operacionais**. O uso em jogos online pode acarretar sanções e banimentos pelos quais o desenvolvedor não assume responsabilidade.
