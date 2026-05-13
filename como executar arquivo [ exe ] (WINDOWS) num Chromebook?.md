# 🛠️ Solução de Erros: Rodando Autoteste (.exe) do Windows no ChromeOS (Chromebook)

Este repositório documenta a solução encontrada para executar um script de autoteste acadêmico compilado para Windows (`.exe`) dentro do ambiente Linux integrado do ChromeOS, permitindo a validação de um projeto de Contador de Caracteres em C.

---

## 🚨 O Problema Original
Ao tentar rodar um executável do Windows (`autotest.exe`) e testar um programa em C no Chromebook, enfrentamos duas barreiras principais:
1. O ChromeOS não executa arquivos `.exe` nativamente.
2. O script de autoteste exige uma estrutura específica de pastas e arquivos de entrada (`.txt`) para validação.

Comando original do Windows:
```bash
autotest.exe ..\bin\Debug\contadores_caracteres.exe
```

---

## 🔬 Diagnóstico do Log do Kernel (Wine)
Durante as tentativas de execução via camada de compatibilidade, os seguintes erros foram identificados e mitigados:

*   **Aviso de Arquitetura:** `it looks like wine32 is missing...` -> Falta de suporte a binários Windows de 32-bits no subsistema Linux de 64-bits.
*   **Erros OLE/RPC:** `0050:err:ole:StdMarshalImpl_MarshalInterface Failed...` -> Falhas de inicialização do ambiente virtual causadas pela falta das dependências de sistema do Windows.
*   **Erro Crítico do Script:** `Error: no test files are found` -> O `autotest.exe` iniciou com sucesso através do Wine, mas falhou porque não localizou os arquivos `.txt` com os cenários de teste na pasta de execução atual (`.`).

---

## 🚀 Passo a Passo da Solução (O que funcionou)

### 1. Preparação do Ambiente no Terminal Linux
Atualize os repositórios e instale o Wine (certificando-se de que os arquivos de teste `.txt` enviados pelo professor estejam no mesmo diretório de execução):
```bash
sudo apt update
sudo apt install wine -y
```

### 2. Compilação Cruzada (Cross-Compilation) da Aplicação C
Como o testador do professor avalia um binário Windows, compilamos o código C nativamente no Linux gerando um formato `.exe` compatível, usando o `mingw-w64`:
```bash
# Instalação do compilador cruzado
sudo apt install mingw-w64 -y

# Compilando o arquivo C para executável Windows
x86_64-w64-mingw32-gcc seu_codigo.c -o contadores_caracteres.exe
```

### 3. Estruturação do Diretório
O `autotest.exe` busca o executável do projeto dentro de caminhos específicos (`bin/Debug`). A estrutura de pastas foi replicada manualmente:
```bash
# Criação da árvore de diretórios exigida
mkdir -p bin/Debug

# Movendo o binário compilado para a pasta correta
mv contadores_caracteres.exe bin/Debug/
```

### 4. Execução Final com Sucesso
Certificando que os arquivos `.txt` de teste do professor foram movidos para a raiz (`.`), o comando final corrigido com barras padrão Unix executou com sucesso:
```bash
wine autotest.exe bin/Debug/contadores_caracteres.exe
```

---

## 💡 Aprendizados Chave
*   **Multiplataforma Forçada:** O Wine consegue interpretar chamadas de sistema Windows dentro do Chromebook, desde que a estrutura de caminhos (`\ ` vs `/`) e arquivos dependentes (`.txt`) seja estritamente respeitada.
*   **Independência de Ambiente:** Não é necessário migrar para o Windows para validar projetos da faculdade; o terminal Linux do ChromeOS é robusto o suficiente usando as ferramentas certas.
