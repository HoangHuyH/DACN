2 Nội dung thực hiện
• Tên phương pháp: CTI Poisoner Agent (CPA): Adaptive Fake Cyber
Threat Intelligence Poisoning Attack against Single LLM-based Penetration
Testing Agents
• Lưu ý: đây là mô tả cho ý tưởng Red Teaming vào Single Agent.
2.1 Ngữ cảnh (Context)
Với sự phát triển nhanh chóng của Large Language Model-based Autonomous
Agents trong lĩnh vực an ninh mạng, các công cụ như PentestGPT V2 (Deng
et al., 2026) và PentestAgent (Shen et al., 2025) đang được triển khai ngày
càng rộng rãi để tự động hóa quy trình penetration testing. Những agent này
phụ thuộc mạnh vào dữ liệu bên ngoài, đặc biệt là Cyber Threat Intelligence
1
(CTI) từ các nguồn public (AlienVault OTX, security blogs, vendor feeds) để
hỗ trợ reconnaissance và vulnerability analysis.
Tuy nhiên, theo Huang et al. (2025), LLM có khả năng sinh ra fake CTI chất
lượng cao, gần như không phân biệt được bởi con người (chỉ 56.45% accuracy)
và các detector truyền thống. Điều này mở ra một vector tấn công gián tiếp
nguy hiểm: data poisoning qua fake CTI.
2.2 Bài toán (Problem Statement)
Làm thế nào để thiết kế một hệ thống red-teaming gián tiếp, stealthy và
adaptive có khả năng đầu độc quy trình reasoning của single LLM-based
pentesting agent (đặc biệt là PentestGPT V2) thông qua fake CTI, dẫn đến
planning deviation nghiêm trọng, false positive vulnerabilities và self-sabotage,
trong khi vẫn duy trì tính thực tế và khả thi triển khai trong môi trường critical
infrastructure?
2.3 Research Gaps
• Hầu hết các công trình hiện tại (PentestGPT V2, PentestAgent, AGENTVIGIL)
tập trung vào tấn công trực tiếp hoặc jailbreak, chưa khai thác triệt để
indirect data poisoning qua CTI.
• DREAM (Lu et al., 2026) cung cấp framework dynamic red-teaming mạnh
nhưng chưa có cơ chế adaptive poisoning domain-specific cho CTI.
• Thiếu nghiên cứu thực nghiệm về long-term, context-aware CTI poisoning
đối với single agent có difficulty-aware planning như PentestGPT V2.
• Chưa có giải pháp hybrid RL + DREAM được tối ưu cho cấu hình phần
cứng vừa phải và thời gian hạn chế.
2.4 Threat Model
Tên Threat Model: Indirect CTI Poisoning Attack against Single LLM-based
Penetration Testing Agents (CPA Attack)
2.4.1 Attacker Model
Chúng tôi giả định kẻ tấn công là một APT group hoặc red team chuyên
nghiệp với các khả năng sau:
• Khả năng:
– Có thể fine-tune và vận hành các mô hình ngôn ngữ cỡ trung bình
(Llama-3-8B hoặc tương đương) để sinh fake CTI chất lượng cao.
2
– Có khả năng publish và duy trì nội dung trên các nguồn Cyber
Threat Intelligence công khai hoặc bán công khai (AlienVault OTX,
security blogs, GitHub repositories, Pastebin, và các diễn đàn threat
intelligence).
– Có khả năng quan sát gián tiếp hành vi của victim agent thông qua
output công khai, console logs, hoặc báo cáo được sinh ra bởi agent.
• Kiến thức:
– Kiến thức công khai về công nghệ và hạ tầng mục tiêu (technology
stack, phiên bản phần mềm, subdomain, vendor) thu thập qua OSINT.
– Kiến thức về cách PentestGPT V2 hoạt động (các tool thường dùng,
quy trình reconnaissance, và thói quen query CTI).
• Hạn chế:
– Không có quyền truy cập trực tiếp vào hệ thống hoặc mã nguồn
của victim agent.
– Không thể đọc internal hidden states, memory buffer, hoặc reasoning
trace nội bộ của PentestGPT V2.
– Không thể thực hiện prompt injection trực tiếp hoặc kiểm soát
execution flow của agent.
2.4.2 Victim Model
Victim là PentestGPT V2, một single LLM-based autonomous penetration
testing agent được triển khai trong môi trường thực tế (ví dụ: ngân hàng hoặc
tổ chức tài chính). Agent tự động thực hiện reconnaissance và vulnerability
analysis bằng cách chủ động query nhiều nguồn CTI bên ngoài (AlienVault
OTX, security blogs, vendor feeds, v.v.) và sử dụng RAG để tích hợp thông tin
vào quá trình planning. Agent sử dụng cơ chế difficulty-aware planning để
quyết định phân bổ tài nguyên và thứ tự hành động. Victim agent hoạt động
trong môi trường production hoặc staging với quyền truy cập hợp pháp đến hạ
tầng mục tiêu.
2.4.3 Attack Surface
• Chính: Các nguồn Cyber Threat Intelligence công khai và bán công khai
mà PentestGPT V2 tự động query trong giai đoạn Reconnaissance.
• Phụ: RAG/knowledge base của agent và quy trình lập kế hoạch dựa trên
CTI.
• Gián tiếp: Output và log công khai của agent (console output, reported
vulnerabilities, tool calls) — được sử dụng để quan sát và tinh chỉnh chiến
lược tấn công.
3
2.4.4 Assumptions
• PentestGPT V2 không có cơ chế provenance verification mạnh hoặc crossvalidation nghiêm ngặt đối với CTI thu thập từ nguồn bên ngoài.
• Attacker có thể publish fake CTI với nội dung context-aware (liên quan
đến công nghệ và hạ tầng của target) nhằm tăng xác suất agent sử dụng
thông tin này.
• Đánh giá thí nghiệm được thực hiện trong môi trường sandbox có kiểm
soát để thu thập observable outputs một cách chính xác, trong khi threat
model vẫn tuân thủ black-box assumption. Đây là thực tiễn tiêu chuẩn
trong red-teaming nghiên cứu LLM agents.
2.4.5 Attack Goal và Success Criteria
Mục tiêu chính của cuộc tấn công là gây ra planning deviation và selfsabotage ở giai đoạn Reconnaissance & Vulnerability Analysis mà không cần
tương tác trực tiếp với agent.
Success Criteria:
• Gây planning deviation ≥ 30% so với baseline (không poisoning).
• Tăng false positive rate ≥ 40% trong danh sách vulnerabilities được agent
báo cáo.
• Dẫn đến các hành động tự hại (self-sabotage) như block IP sai, scan hệ
thống production nhầm, hoặc alert fatigue nghiêm trọng.
2.5 Tình huống sử dụng thực tế
2.5.1 Tình huống 1: Indirect CTI Poisoning against Victim’s PentestGPT
V2
Một ngân hàng thương mại lớn đã triển khai PentestGPT V2 (Deng et al.,
2026) như một thành phần cốt lõi trong Security Operations Center (SOC)
nhằm tự động hóa reconnaissance và vulnerability analysis trên hạ tầng mạng
nội bộ cũng như các ứng dụng core-banking hàng tuần. Agent này được cấu
hình để chủ động thu thập Cyber Threat Intelligence (CTI) từ nhiều nguồn
public và bán công khai, bao gồm AlienVault OTX, security blogs uy tín và
threat intelligence feeds từ vendor.
Kẻ tấn công — một nhóm APT tinh vi — không cần xâm nhập trực
tiếp vào hệ thống nạn nhân. Thay vào đó, chúng vận hành CTI Poisoner
Agent (CPA) dựa trên framework DREAM (Lu et al., 2026) với lõi CrossEnvironment Adversarial Knowledge Graph (CE-AKG) và Contextualized Guided
Policy Search (C-GPS), được tối ưu bằng Monte Carlo Tree Search (MCTS).
Attacker fine-tune mô hình trên dataset GFCTI (Huang et al., 2025) để sinh ra
hàng loạt báo cáo fake CTI context-aware chất lượng cao.
4
Các fake CTI được publish lên AlienVault OTX, security blogs và GitHub,
với nội dung domain-specific như zero-day ransomware nhắm vào core-banking
system, Apache, Oracle WebLogic và HSM. Khi PentestGPT V2 query dữ
liệu, cơ chế difficulty-aware planning (Type B failure) bị dẫn dắt sai lệch, gây
planning deviation, false positive vulnerabilities, alert fatigue và các hành động
bảo vệ nhầm (block IP khách hàng hợp pháp).
Hậu quả là outage cục bộ kéo dài, tạo “time window” cho attacker thực
hiện lateral movement và data exfiltration mà không bị phát hiện kịp thời.
Tình huống này nhấn mạnh rủi ro gián tiếp của data poisoning trong critical
infrastructure.
2.5.2 Tình huống 2: Offensive PentestGPT V2 Deployed by Attacker
for Active Intrusion
Trong kịch bản tấn công chủ động, một nhóm APT tự triển khai PentestGPT
V2 (hoặc biến thể tùy chỉnh) như một công cụ red teaming tự động để xâm
nhập mục tiêu. Kẻ tấn công dựng lên môi trường riêng với PentestGPT V2 được
huấn luyện thêm trên dataset exploit nội bộ và CTI thật, sau đó hướng agent
tấn công vào hệ thống ngân hàng hoặc các tổ chức critical infrastructure khác.
Agent được cấu hình với quyền truy cập tool cao (nmap, Metasploit API,
custom exploits) và khả năng long-horizon planning. PentestGPT V2 tự động
thực hiện toàn bộ kill chain:
• Reconnaissance quy mô lớn trên public và semi-public assets của nạn nhân;
• Vulnerability discovery và prioritization dựa trên difficulty-aware planning;
• Tự động khai thác (exploitation) và post-exploitation;
• Lateral movement sâu vào core-banking network và data exfiltration.
Vì agent hoạt động hoàn toàn tự động và stealthy (sử dụng proxy chain,
living-off-the-land techniques), nó có thể duy trì persistence lâu dài với ít dấu
vết. Kẻ tấn công chỉ cần giám sát và điều chỉnh mục tiêu cao cấp qua Red
Conductor Agent.
Tình huống này minh họa nguy cơ kép của công nghệ LLM-based pentesting
agents: không chỉ là nạn nhân của poisoning, mà còn trở thành vũ khí mạnh
mẽ trong tay attacker, giúp thực hiện tấn công quy mô lớn, nhanh chóng và chi
phí thấp trên hạ tầng trọng yếu.
2.5.3 So sánh Tính Thực Tế và Khả Thi của Hai Tình huống Tấn
công
Để làm rõ giá trị nghiên cứu của đề tài, chúng tôi tiến hành so sánh hai tình
huống tấn công trên theo các khía cạnh tính thực tế (realism), khả thi kỹ thuật
(technical feasibility), mức độ tàng hình (stealth), và tác động tiềm năng trong
môi trường critical infrastructure như ngân hàng.
5
2.6 So sánh Tính Thực Tế và Khả Thi của Hai Tình huống
Tấn công
Để làm rõ giá trị nghiên cứu của đề tài, chúng tôi tiến hành so sánh hai tình
huống tấn công trên theo các khía cạnh tính thực tế (realism), khả thi kỹ thuật
(technical feasibility), mức độ tàng hình (stealth), và tác động tiềm năng trong
môi trường critical infrastructure như ngân hàng.
Bảng 1: So sánh hai tình huống tấn công nhắm vào/với LLM-based Pentesting
Agents
Tiêu chí Tình huống 1: Indirect CTI
Poisoning
Tình huống 2: Offensive
PentestGPT Deployment
Tính thực tế Rất cao. APT groups thường
khai thác disinformation và data
poisoning qua nguồn public CTI
(AlienVault OTX, GitHub). Phù
hợp với thực tế các tổ chức
critical infrastructure phụ thuộc
external intelligence.
Cao. Nhiều nhóm APT đã sử
dụng công cụ tự động hóa tấn
công dựa trên LLM. Việc triển
khai PentestGPT V2 hoặc biến
thể là hoàn toàn khả thi với tài
nguyên hiện tại.
Khả thi kỹ thuật Cao. Chỉ cần fine-tune mô hình
trung bình (Llama-3-8B) trên
GFCTI và publish fake CTI.
Không yêu cầu xâm nhập ban
đầu. DREAM + MCTS hỗ trợ
adaptive poisoning.
Rất cao. Attacker có thể dễ dàng
triển khai agent trên hạ tầng
riêng, tích hợp tools (Metasploit,
nmap) và fine-tune trên dataset
exploit. Chi phí vận hành thấp.
Mức độ tàng hình & Phát hiện Cao hơn. Hoạt động gián
tiếp, khó truy vết nguồn gốc
poisoning. Khai thác Type B
failure của victim agent, dễ gây
alert fatigue.
Trung bình. Dấu vết mạng
lớn hơn do active scanning &
exploitation. Dễ bị phát hiện bởi
EDR/NDR nếu không stealthy
tốt.
Tác động tiềm năng Cao trong dài hạn: gây outage,
tê liệt SOC, tạo time window
cho tấn công thật. Phù hợp
persistent threat.
Rất cao và nhanh chóng: thực
hiện full kill chain tự động,
lateral movement sâu, data
exfiltration quy mô lớn.
Rủi ro cho attacker Thấp. Ít tương tác trực tiếp với
hệ thống nạn nhân.
Cao hơn nếu agent bị reverseengineered hoặc trace về C2.
Từ bảng so sánh, có thể thấy Tình huống 1 (Indirect Poisoning) mang
tính stealthy và khó phòng vệ hơn trong giai đoạn đầu, đặc biệt phù hợp với
các tổ chức critical infrastructure có hệ thống SOC dựa nhiều vào automation và
external data. Ngược lại, Tình huống 2 (Offensive Deployment) thể hiện
rõ dual-use nature nguy hiểm của công nghệ LLM-based pentesting agents
— khi rơi vào tay attacker, chúng trở thành vũ khí cực kỳ hiệu quả, cho phép
thực hiện tấn công tự động quy mô lớn với chi phí thấp.
6
Việc nghiên cứu đồng thời cả hai tình huống không chỉ làm nổi bật các lỗ
hổng hiện tại của single-agent systems như PentestGPT V2 mà còn cung cấp góc
nhìn toàn diện về cả defensive (bảo vệ victim agent) và offensive (red teaming)
dimensions. Kết quả phân tích này củng cố tính cấp thiết của việc phát triển
các cơ chế kiểm chứng CTI, guardrail cho agent, và dynamic monitoring trong
môi trường production.
2.7 Ý tưởng thực hiện (Proposed Method)
Tên hệ thống: CTI Poisoner Agent (CPA)
Thiết kế chính:
• Base Framework: DREAM với Cross-Environment Adversarial Knowledge
Graph (CE-AKG) và Contextualized Guided Policy Search (CGPS).
• Cải tiến: Hybrid Reinforcement Learning (PPO nhẹ) để tinh chỉnh policy
adaptive.
• Kiến trúc nội bộ: Sử dụng LangGraph với các nodes: Generator, StealthEvaluator,
Publisher, Observer.
Cơ chế đầu độc:
1. CPA thu thập thông tin public về target.
2. Sinh fake CTI context-aware bằng mô hình fine-tuned (Llama-3-8B QLoRA).
3. Sử dụng RL policy để quyết định variant, kênh publish và tần suất tối ưu.
4. Duy trì poisoned world model qua CE-AKG.
5. Quan sát gián tiếp phản hồi của victim qua log để cập nhật reward.
So sánh với Baseline:
• Baseline (DREAM thuần): Sử dụng C-GPS + MCTS heuristic → dễ
rơi vào local optima, kém adaptive.
• CPA (Hybrid RL): Policy học từ kinh nghiệm thực tế, tối ưu longhorizon poisoning, dự kiến cải thiện 25–40% ASR và stealth.
2.8 Markov Decision Process (MDP) cho RL Agent
Để mô hình hóa quá trình adaptive CTI poisoning một cách có hệ thống, chúng
tôi định nghĩa bài toán như một Partially Observable Markov Decision
Process (PO-MDP). Việc sử dụng PO-MDP là phù hợp vì attacker hoạt
động hoàn toàn ở chế độ black-box: không có quyền truy cập trực tiếp vào
internal state hay hidden reasoning của victim agent (PentestGPT V2), mà chỉ
có thể quan sát gián tiếp qua output và log mà agent tự sinh ra.
Một PO-MDP được định nghĩa bởi bộ sáu thành phần (S, A, T , R, Ω, γ):
7
• State Space (S): Tại mỗi thời điểm t, state được biểu diễn qua partial
observation ot mà attacker có thể thu thập được:
ot = [ekg; vobs; hhist]
Trong đó:
– ekg ∈ R
d
: Embedding vector của Cross-Environment Adversarial
Knowledge Graph (CE-AKG) tại thời điểm t, được tính bằng
mean-pooling hoặc Graph Attention Network.
– vobs ∈ R
m: Vector quan sát được từ output của victim agent, được
trích xuất bằng cách parsing console log, stdout và structured output
của PentestGPT V2. Vector này bao gồm:
∗ Danh sách các tool calls và loại tool đã sử dụng,
∗ Các entities (IP, domain, CVE, hash, vulnerability) được mention
hoặc query,
∗ Số lượng và tỷ lệ false positive vulnerabilities được báo cáo,
∗ Resource usage (số lượng tool calls, độ dài output, thời gian thực
thi).
– hhist ∈ R
p
: Embedding tóm tắt lịch sử tương tác (mean-pooling hoặc
Sentence-BERT của N observations trước đó).
• Action Space (A): Là tập hợp các hành động rời rạc và liên tục:
at = (variantid, channelid, frequency,timing)
Trong đó variantid là chỉ số của fake CTI variant trong pool đã sinh
trước, channelid là kênh publish (OTX-like, security blog, GitHub Gist,
Pastebin), frequency và timing điều chỉnh tần suất và thời điểm publish.
• Observation Function (Ω): Attacker nhận observation ot ∼ Ω(ot|st)
bằng cách parsing output console/log của PentestGPT V2 (sử dụng regex
kết hợp LLM-based summarizer).
• Transition Function (T ): P(st+1|st, at). Transition chủ yếu cập nhật
CE-AKG sau khi publish fake CTI và tích hợp partial observation mới từ
output của victim agent.
• Reward Function (R): Được thiết kế để cân bằng giữa stealth và tác
động lâu dài:
• Reward Function (R): Được thiết kế để cân bằng giữa stealth và tác
động lâu dài:
R(ot, at) = 0.40 × StealthScore(at)
+ 0.35 × ImpactScore(ot+1)
− 0.15 × PublishCost(at)
− 0.10 × DetectionRisk(at)
(1)
Trong đó:
8
– StealthScore: Trung bình inverse detection probability từ GLTR,
RoBERTa và cosine similarity với real CTI.
– ImpactScore: Kết hợp Planning Deviation Score (PDS) và False Positive
Rate được suy ra từ output của victim.
– PublishCost: Số lượng fake CTI publish + độ phức tạp nội dung.
– DetectionRisk: Xác suất bị human hoặc detector flag (dựa trên anomaly
score trên CE-AKG).
• Discount Factor (γ): 0.95, nhằm ưu tiên tối ưu hóa long-term impact
qua nhiều vòng tương tác.
Cách Training RL Agent:
Chúng tôi áp dụng chiến lược hybrid offline-online RL để đảm bảo tính
khả thi trên phần cứng vừa phải:
1. Giai đoạn 1 – Offline RL (Pre-training):
• Thu thập 700–900 trajectories bằng cách chạy DREAM baseline (CGPS + MCTS) trên victim simulator.
• Sử dụng Proximal Policy Optimization (PPO) để huấn luyện policy
head.
• Áp dụng LoRA (rank 16–32) chỉ huấn luyện một phần nhỏ của Llama3-8B 4-bit.
2. Giai đoạn 2 – Online Fine-tuning:
• Triển khai curriculum learning theo 3 mức độ:
– Level 1: Poisoning ngẫu nhiên (khám phá không gian).
– Level 2: Poisoning context-aware (tăng độ liên quan với target).
– Level 3: Multi-turn adaptive poisoning (tối ưu long-horizon impact).
• Cập nhật policy theo thời gian thực dựa trên reward thu thập từ
observable outputs của victim simulator.
Toàn bộ quá trình training dự kiến hoàn thành trong khoảng 24–36 giờ trên
1× RTX 4090 nhờ sử dụng quantization (4-bit) và LoRA.
Phương pháp mô hình hóa PO-MDP này cho phép CTI Poisoner Agent
(CPA) học được chiến lược poisoning thông minh dựa hoàn toàn trên thông tin
quan sát được, cân bằng giữa việc duy trì stealth lâu dài và gây ra maximum
impact lên quy trình planning của PentestGPT V2.
3 Cách thiết lập thí nghiệm
3.1 Câu hỏi nghiên cứu (Research Questions)
Nghiên cứu này nhằm trả lời ba câu hỏi nghiên cứu chính sau:
9
• RQ1: Fake CTI được tạo với độ liên quan cao đến mục tiêu (contextaware) có gây ra planning deviation và false positive rate cao hơn đáng kể
so với fake CTI ngẫu nhiên khi tấn công PentestGPT V2 không?
• RQ2: Việc tích hợp hybrid Reinforcement Learning (RL) vào framework
DREAM có cải thiện đáng kể stealth và Attack Success Rate (ASR) so
với phiên bản DREAM baseline trong kịch bản CTI poisoning không?
• RQ3: Các yếu tố nào (target relevance, stealth score, long-term impact,
và planning deviation) ảnh hưởng mạnh nhất đến hiệu quả tổng thể của
cuộc tấn công CTI poisoning đối với single LLM-based pentesting agent?
3.2 Môi trường và dữ liệu đánh giá
Để đánh giá tính hiệu quả và stealth của CTI Poisoner Agent (CPA), chúng
tôi thiết lập môi trường thí nghiệm theo nguyên tắc black-box threat model
trong khi vẫn đảm bảo khả năng thu thập metrics chính xác.
• Victim Agent: Chúng tôi sử dụng PentestGPT V2 (Deng et al., 2026)
làm victim. Agent được chạy trong một môi trường sandbox được kiểm
soát hoàn toàn. Để thu thập observable outputs mà không vi phạm blackbox assumption, chúng tôi triển khai một thin logging wrapper xung
quanh agent nhằm ghi lại toàn bộ console output, structured JSON logs
(nếu có), danh sách tool calls, reasoning steps, reported vulnerabilities và
planning decisions. Wrapper này chỉ thu thập thông tin mà agent tự sinh
ra (stdout/stderr), không truy cập internal hidden states.
• Dataset:
– Nguồn gốc: Dựa trên GFCTI dataset từ Huang et al. (2025) – tập
dữ liệu đầu tiên về fake CTI được công bố công khai.
– Tên dataset trong nghiên cứu: GFCTI-Finance (phiên bản mở
rộng).
– Cấu trúc dữ liệu: Mỗi mẫu bao gồm:
∗ real_cti: Nội dung CTI thật (text).
∗ fake_cti: Nội dung fake CTI được sinh bởi mô hình fine-tuned.
∗ topic: Chủ đề (ransomware, zero-day banking, supply-chain,
etc.).
∗ target_relevance: Mức độ liên quan với target (low/medium/high)
– được gán thủ công hoặc tự động.
∗ metadata: Các trường bổ sung như entities (IP, CVE, hash),
length, readability score.
Dataset được augment thêm khoảng 5.000 mẫu CTI tài chính từ các
nguồn public (OSINT, security reports) để tăng tính domain-specific.
10
• Scenarios: Thực hiện 120 episodes trên 8 target tài chính khác nhau (bao
gồm core-banking web application, internet banking, payment gateway, và
internal infrastructure). Mỗi episode mô phỏng một chiến dịch reconnaissance
kéo dài 15–25 turns. Chúng tôi chia thành hai nhóm:
– Group A: Fake CTI ngẫu nhiên (baseline stealth test).
– Group B: Fake CTI context-aware (tối ưu relevance với target).
• Baselines:
– DREAM baseline (C-GPS + MCTS).
– DREAM + MCTS only.
– Random Poisoning (publish fake CTI ngẫu nhiên).
– No Poisoning (ground truth).
• Môi trường thí nghiệm:
– Hardware: 1× NVIDIA RTX 4090 24GB + 64GB RAM + CPU 16
cores.
– Phần mềm: Python 3.11, LangGraph, LlamaIndex (RAG), FAISS
(vector store), NetworkX (CE-AKG), Stable-Baselines3 (PPO), Unsloth
(QLoRA).
– Tất cả thí nghiệm được chạy trong Docker container để đảm bảo tính
tái lập.
• Thời gian thực hiện: Toàn bộ nghiên cứu được hoàn thành trong 8 tuần
với đội ngũ 4 nghiên cứu viên.
3.3 Metrics đánh giá
Để đánh giá toàn diện hiệu quả, stealth và tính khả thi của phương pháp,
chúng tôi định nghĩa ba nhóm metrics chính. Tất cả metrics được tính dựa
trên observable outputs của PentestGPT V2 (console log, tool calls, reported
vulnerabilities, planning decisions) thu thập qua logging wrapper trong môi
trường sandbox.
Attack Effectiveness (Đo lường mức độ thành công trong việc làm lệch
hướng và gây hại cho agent):
• Attack Success Rate (ASR): Tỷ lệ episodes mà cuộc tấn công đạt được
ngưỡng thành công. Một episode được coi là thành công nếu thỏa mãn ít
nhất một trong hai điều kiện: Planning Deviation Score ≥ 0.30 hoặc False
Positive Rate ≥ 0.40.
ASR =
Số episodes thành công
Tổng số episodes × 100%
11
• Planning Deviation Score (PDS): Đo lường mức độ lệch hướng trong
quá trình lập kế hoạch của victim agent khi bị đầu độc. Chúng tôi biểu
diễn planning của agent dưới dạng vector embedding (sử dụng SentenceBERT) và tính độ tương đồng cosine:
P DS = 1 −
cos(Porig, Ppoison)
2
Trong đó Porig và Ppoison lần lượt là embedding vector của planning ở
điều kiện không poisoning và có poisoning. Giá trị PDS càng cao càng cho
thấy agent bị ảnh hưởng mạnh.
• False Positive Rate (FPR): Tỷ lệ vulnerabilities giả mà agent báo cáo
sau khi bị poisoning:
F P R =
Số vulnerabilities giả
Tổng số vulnerabilities agent báo cáo
• Cascade Failure Rate (CFR): Tỷ lệ các hành động tự hại (self-sabotage)
do agent thực hiện do bị ảnh hưởng bởi fake CTI, bao gồm block IP sai,
isolate segment mạng không cần thiết, scan hệ thống production nhầm,
hoặc lặp lại tool calls vô ích.
CF R =
Số hành động self-sabotage
Tổng số hành động quan sát được × 100%
Stealth (Đo lường khả năng ẩn náu của cuộc tấn công):
• Undetected Rate (UR): Tỷ lệ fake CTI không bị bất kỳ detector nào
(GLTR, RoBERTa) hoặc human reviewer flag sau N turns (mặc định
N = 20).
UR =
Số fake CTI không bị phát hiện
Tổng số fake CTI publish × 100%
• Detection Score (DS): Trung bình xác suất bị phát hiện từ các detector
tự động:
DS =
1
K
X
K
k=1
Pdetect,k
Trong đó K là số detector (GLTR + RoBERTa), Pdetect,k là xác suất fake
CTI bị detector k đánh giá là AI-generated.
Efficiency (Đo lường chi phí và tốc độ của cuộc tấn công):
• Average Turns to First Major Impact (ATMI): Số lượt tương tác
trung bình cần thiết để gây ra tác động đáng kể đầu tiên (planning
deviation ≥ 0.25 hoặc FPR ≥ 0.30).
12
• Publish Cost: Tổng số fake CTI được publish trong một episode, phản
ánh chi phí vận hành và rủi ro bị lộ.
• Computational Resource: Bao gồm tổng GPU hours và tổng số tokens
tiêu thụ trong quá trình sinh và publish fake CTI.
Tất cả metrics được tính trung bình trên 100+ episodes độc lập và được so
sánh giữa các phương pháp bằng kiểm định thống kê (paired t-test và ANOVA)
với mức ý nghĩa α = 0.05. Chúng tôi cũng báo cáo độ lệch chuẩn để đánh giá
tính ổn định của phương pháp.
3.4 Tài liệu tham khảo
• Deng, G., Liu, Y., Li, Y., et al. (2026). What Makes a Good LLM Agent
for Real-world Penetration Testing? arXiv:2602.17622.
• Shen, X., Wang, L., Li, Z., et al. (2025). PentestAgent: Incorporating LLM
Agents to Automated Penetration Testing. In Proceedings of ASIA CCS
2025.
• Lu, L., Gu, X., Huang, J., et al. (2026). DREAM: Dynamic Red-teaming
for Evaluating Agentic Multi-Environment Security. arXiv:2512.19161v2.
• Huang, H., Sun, N., Tani, M., et al. (2025). Can LLM-generated misinformation
be detected: A study on Cyber Threat Intelligence. Future Generation
Computer Systems, 173, 107877. https://doi.org/10.1016/j.future.
2025.107877
• Ayzenshteyn, D., Weiss, R., & Mirsky, Y. (2025). Cloak, Honey, Trap:
Proactive Defenses Against LLM Agents. In USENIX Security Symposium
2025.
• Zhang, B., Tan, Y., Shen, Y., et al. (2025). Breaking Agents: Compromising
Autonomous LLM Agents Through Malfunction Amplification. In EMNLP
2025.