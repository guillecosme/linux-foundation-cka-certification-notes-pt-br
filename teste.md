# O Caminho para Distinguished Engineer: Plano de Estudo de 12 Meses
**Objetivo:** Domínio de Linux, Redes, Arquitetura Distribuída e Performance (Nível FAANG).
**Carga Horária Estimada:** 15-20h semanais.

---

## 📚 A Lista de Leitura (O'Reilly)

### Fase 1: Fundações (Kernel & Código)
1. **How Linux Works, 3rd Ed** (Brian Ward)
2. **Grokking Algorithms** (Aditya Bhargava)
3. **Fluent Python, 2nd Ed** (Luciano Ramalho)
4. **Container Security** (Liz Rice)
5. **Working Effectively with Legacy Code** (Michael Feathers)

### Fase 2: Redes Profundas (Protocolos)
6. **Packet Guide to Core Network Protocols** (Bruce Hartpence)
7. **DNS and BIND, 5th Ed** (Liu & Albitz)
8. **TCP/IP Illustrated, Vol 1: The Protocols** (Fall & Stevens) - *A Bíblia*
9. **Practical Packet Analysis, 3rd Ed** (Chris Sanders)
10. **Bulletproof SSL and TLS** (Ivan Ristić)
11. **Load Balancing in the Cloud** (Derek DeJonghe)

### Fase 3: Arquitetura & Escala
12. **Designing Data-Intensive Applications** (Martin Kleppmann) - *O Divisor de Águas*
13. **Software Architecture: The Hard Parts** (Ford, Richards et al.)
14. **Site Reliability Engineering** (Google Team)

### Fase 4: Distinguished Level (Kernel Power & Liderança)
15. **Systems Performance, 2nd Ed** (Brendan Gregg)
16. **Learning eBPF** (Liz Rice)
17. **BPF Performance Tools** (Brendan Gregg) - *Referência*
18. **High Performance Browser Networking** (Ilya Grigorik) - *Leitura complementar*
19. **Staff Engineer** (Will Larson)

---

## 🗓️ O Cronograma (Sprints Trimestrais)

### 🟢 Trimestre 1: A Fundação de Aço (OS, Code & Algoritmos)
*Foco: Parar de ver o computador como uma "caixa mágica".*

| Mês | Livros Focados | Estratégia de Leitura | Lab Prático Obrigatório |
| :--- | :--- | :--- | :--- |
| **1** | *How Linux Works*<br>*Grokking Algorithms* | **Linux:** Caps 3, 4, 6 (Devices, Disks, Boot).<br>**Algos:** Big O e Hash Maps. | Criar VM Linux "crua" na AWS. Formatar/montar discos via CLI (`fdisk`, `mkfs`) sem Google. |
| **2** | *Fluent Python* (Pt 1)<br>*Container Security* | **Py:** Data Structures.<br>**Container:** Caps 2-4 (Namespaces/Cgroups). | Script Python usando `os.fork()` ou criar container manual com `unshare` no Linux. |
| **3** | *Fluent Python* (Pt 2)<br>*Legacy Code* | **Py:** Control Flow.<br>**Legacy:** Caps 1-4 (Seams & Testing). | Refatorar um script legado aplicando testes unitários (`pytest`) e mocks. |

### 🟡 Trimestre 2: A Rede Profunda (O Pesadelo do TCP)
*Foco: Provar a causa raiz de problemas de rede com dados.*

| Mês | Livros Focados | Estratégia de Leitura | Lab Prático Obrigatório |
| :--- | :--- | :--- | :--- |
| **4** | *Packet Guide*<br>*DNS and BIND* | **DNS:** Caps 2, 4, 10 (Recursão, TTL, Troubleshooting). | Configurar servidor BIND e fazer trace de resolução DNS com `dig +trace`. |
| **5** | *TCP/IP Illustrated*<br>*Practical Packet Analysis* | **TCP:** Caps 13 (Handshake), 14-16 (Congestion/Windows). | Wireshark: Analisar um PCAP de upload lento. Identificar Retransmissões e Janela Zero. |
| **6** | *Bulletproof SSL/TLS*<br>*Load Balancing* | **SSL:** PKI e Chain of Trust.<br>**LB:** L4 vs L7. | Configurar NGINX com cert auto-assinado e capturar o Handshake TLS no Wireshark. |

### 🔴 Trimestre 3: Arquitetura de Escala
*Foco: Trade-offs de Sistemas Distribuídos.*

| Mês | Livros Focados | Estratégia de Leitura | Lab Prático Obrigatório |
| :--- | :--- | :--- | :--- |
| **7** | *Designing Data-Intensive Apps* (Parte 1) | Caps 1-4: Storage Engines (SSTables, B-Trees). | Benchmark de leitura/escrita: Postgres vs DynamoDB com grande volume de dados. |
| **8** | *Designing Data-Intensive Apps* (Parte 2) | Caps 5-7: Replicação e Consistência. | Simular "Split Brain" em cluster de DB (cortar rede) e testar gravação. |
| **9** | *Arch: The Hard Parts*<br>*SRE Book* | Granularidade, Acoplamento e SLOs. | Escrever um "Design Doc" de migração definindo SLOs de latência e disponibilidade. |

### 🟣 Trimestre 4: Wizard Level (Kernel & Liderança)
*Foco: Observabilidade extrema e Influência.*

| Mês | Livros Focados | Estratégia de Leitura | Lab Prático Obrigatório |
| :--- | :--- | :--- | :--- |
| **10** | *Systems Performance* | Caps 1-3 (USE Method), 6 (CPU), 7 (Mem), 10 (Net). | Diagnosticar gargalo em servidor sob estresse (`stress`) usando `vmstat`, `iostat`, `sar`. |
| **11** | *Learning eBPF*<br>*BPF Tools* | Entender a tecnologia e rodar exemplos. | Usar `bcc-tools` (`execsnoop`, `biolatency`) para ver o Kernel em tempo real. |
| **12** | *Staff Engineer* | Sponsorship e Gestão de Tempo. | Escrever proposta de projeto ("Staff Project") para o próximo ano. |

---

## 🏆 Regras de Ouro
1. **Lab > Leitura:** Se estiver cansado de ler, vá para o terminal.
2. **Não Trave:** Pule a matemática complexa do TCP na primeira leitura. Entenda a mecânica.
3. **Consistência:** 1h por dia > 10h no domingo.
