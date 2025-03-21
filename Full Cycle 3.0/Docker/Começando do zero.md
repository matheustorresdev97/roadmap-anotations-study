### Introdução

### O que são Containers?
Um container é um padrão de unidade de software que empacota código e todas as dependêncais de uma aplicação fazendo que a mesma seja executada rapidamente de forma confiável de um ambiente computacional para o outro.
Containers são unidades leves e portáteis de software que incluem tudo o que um aplicativo precisa para ser executado, como código, bibliotecas, dependências e configurações. Eles funcionam como ambientes isolados que garantem que o software opere de maneira consistente, independentemente de onde esteja sendo executado.
As principais características dos containers incluem:

1. Isolamento: executam de forma isolada no mesmo sistema operacional, sem interferir em outros containers ou no sistema host
2. Portabilidade: funcionam consistentemente em qualquer ambiente (desenvolvimento, teste, produção)
3. Eficiência: são muito mais leves que máquinas virtuais, pois compartilham o kernel do sistema operacional host
4. Escalabilidade: podem ser facilmente replicados para lidar com aumento de demanda
5. Imutabilidade: são criados a partir de imagens imutáveis, garantindo consistência

### Como funcionam os Containers ?


![[Captura de tela de 2025-03-20 19-41-11.png]]

![[Captura de tela de 2025-03-20 19-42-35.png]]


**Namespaces**: São uma funcionalidade do kernel Linux que proporciona isolamento entre processos. Cada container recebe seus próprios namespaces para:

- PID (Process ID): Isola os processos
- NET (Network): Isola interfaces de rede
- MNT (Mount): Isola sistemas de arquivos
- UTS (Unix Timesharing System): Isola hostname e domínio
- IPC (Inter-Process Communication): Isola comunicação entre processos
- USER: Isola usuários e grupos


![[Captura de tela de 2025-03-20 19-32-23.png]]


![[Captura de tela de 2025-03-20 19-32-50.png]]

![[Captura de tela de 2025-03-20 19-33-31.png]]



**Control Groups (cgroups)**: Limitam e monitoram recursos do sistema que um container pode usar:

- CPU
- Memória
- Disco I/O
- Rede
- Garantem que um container não consuma todos os recursos da máquina host

![[Captura de tela de 2025-03-20 19-34-13.png]]


![[Captura de tela de 2025-03-20 19-34-46.png]]



**Sistema de Arquivos em Camadas**:

- Containers utilizam sistemas de arquivos em camadas (como OverlayFS)
- A imagem base é somente leitura
- Uma camada de escrita é adicionada quando o container é executado
- Múltiplos containers podem compartilhar as mesmas camadas base, economizando espaço

![[Captura de tela de 2025-03-20 19-35-29.png]]


![[Captura de tela de 2025-03-20 19-36-14.png]]


**Imagens de Container**:

- São pacotes somente leitura com o código da aplicação, bibliotecas, dependências e configurações
- Servem como templates para criar containers
- São construídas usando arquivos de definição (como Dockerfile no Docker)


![[Captura de tela de 2025-03-20 19-38-45.png]]


![[Captura de tela de 2025-03-20 19-40-07.png]]


![[Captura de tela de 2025-03-20 19-42-19.png]]




### Ciclo de Vida de um Container

1. **Criação**: Um container é criado a partir de uma imagem
2. **Execução**: O container inicia o processo principal definido na imagem
3. **Pausa/Continuação**: Containers podem ser pausados e depois continuados
4. **Parada**: O container é parado, mas seu estado é preservado
5. **Remoção**: O container e seus dados são removidos


![[Captura de tela de 2025-03-20 19-43-37.png]]


## Instalando Docker

https://docs.docker.com/engine/install/debian/


### Documentação Oficial
https://docs.docker.com/
