+++

title = "Blog 5"

weight = 5

chapter = false

pre = "<b> 3.5. </b>"

+++

# Ra mắt Strands Agents 1.0: Đơn giản hóa điều phối Multi-Agent cho môi trường production

**Ryan Coleman, Belle Guttman**  

**Ngày:** 15/07/2025  

**Chủ đề:** Amazon Machine Learning, Announcements, Artificial Intelligence, Open Source

---

Hôm nay, chúng tôi vui mừng giới thiệu **Strands Agents SDK phiên bản 1.0**, đánh dấu một cột mốc quan trọng trong hành trình giúp việc xây dựng các agent AI trở nên **đơn giản, đáng tin cậy và sẵn sàng cho môi trường production**.

[Strands Agents](https://github.com/strands-agents/sdk-python) là một SDK mã nguồn mở theo hướng *model-driven*, cho phép bạn xây dựng và vận hành các agent AI chỉ với vài dòng code. SDK này có thể mở rộng từ những agent đơn lẻ đơn giản cho đến các hệ thống **multi-agent phức tạp**, đồng thời hỗ trợ đầy đủ từ môi trường phát triển cục bộ đến triển khai production.

Kể từ khi ra mắt bản preview vào tháng 5/2025, Strands đã đạt:

- Hơn **2.000 sao GitHub**

- Hơn **150.000 lượt tải** trên PyPI

Phiên bản **Strands Agents 1.0** mang mức độ đơn giản tương tự cho các hệ thống multi-agent, thông qua:

- Bốn **primitive chính** cho multi-agent orchestration  

- Hỗ trợ **Agent-to-Agent (A2A)** protocol  

- Quản lý session bền vững cho production  

- Cải thiện hỗ trợ **bất đồng bộ (async)** toàn diện  

- Mở rộng hỗ trợ nhiều nhà cung cấp mô hình AI

Các ví dụ code hoàn chỉnh có tại [https://strandsagents.com](https://strandsagents.com).

---

## Đơn giản hóa mô hình Multi-Agent

Hệ thống multi-agent cho phép các agent chuyên biệt phối hợp với nhau để giải quyết những bài toán phức tạp mà một agent đơn lẻ không thể xử lý hiệu quả. Strands 1.0 giới thiệu bốn primitive trực quan, giúp điều phối nhiều agent trở nên tự nhiên và dễ mở rộng.

### 1. Agents-as-Tools: Ủy quyền theo cấp bậc

Mô hình **agents-as-tools** cho phép biến các agent chuyên biệt thành công cụ để agent điều phối gọi tới. Agent điều phối giữ quyền kiểm soát tổng thể, đồng thời chủ động tham vấn các "chuyên gia" khi cần --- tương tự cách làm việc trong các nhóm con người.

```python

from strands import Agent, tool

from strands_tools import calculator, file_write, python_repl, journal

@tool

def web_search(query: str) -> str:

    return "Dummy web search results here!"

research_analyst_agent = Agent(

    system_prompt="You are a research specialist",

    tools=[web_search, calculator, file_write, python_repl]

)

travel_advisor_agent = Agent(

    system_prompt="You are a travel expert",

    tools=[web_search, journal]

)

@tool

def research_analyst(query: str) -> str:

    return str(research_analyst_agent(query))

@tool

def travel_advisor(query: str) -> str:

    return str(travel_advisor_agent(query))

executive_assistant = Agent(

    tools=[research_analyst, travel_advisor]

)
```

2\. Handoffs: Chuyển giao quyền kiểm soát rõ ràng

Handoffs cho phép agent chủ động chuyển trách nhiệm sang con người khi gặp yêu cầu vượt ngoài phạm vi xử lý, trong khi vẫn giữ nguyên toàn bộ ngữ cảnh hội thoại.

```python
from strands import Agent

from strands_tools import handoff_to_user

agent = Agent(

    system_prompt="Ask users for more info when needed",

    tools=[handoff_to_user]

)
```

Cơ chế này rất phù hợp với các hệ thống hỗ trợ khách hàng, nơi agent đóng vai trò tuyến đầu và con người xử lý các trường hợp đặc biệt.

3\. Swarms: Nhóm agent tự tổ chức

Swarms cho phép nhiều agent phối hợp động thông qua bộ nhớ dùng chung. Các agent có thể tự tổ chức, bổ trợ lẫn nhau để cho ra kết quả tổng hợp tốt nhất.

```python
from strands import Agent

from strands.multiagent import Swarm

researcher = Agent(name="researcher")

analyst = Agent(name="analyst")

writer = Agent(name="writer")

market_research_team = Swarm([researcher, analyst, writer])

result = market_research_team("Create a report on AI history")
```

4\. Graphs: Quy trình xác định và kiểm soát

Graphs cho phép định nghĩa rõ ràng luồng xử lý, điều kiện rẽ nhánh và các điểm kiểm soát chất lượng, phù hợp cho các workflow có quy tắc nghiệp vụ nghiêm ngặt.

```python
from strands.multiagent import GraphBuilder

builder = GraphBuilder()

builder.set_entry_point("analyze")

graph = builder.build()
```

Các mô hình này có thể kết hợp linh hoạt: agents-as-tools trong swarms, swarms trong graphs, hoặc graphs điều phối swarms.

Hệ thống Multi-Agent với A2A

Strands 1.0 hỗ trợ đầy đủ Agent-to-Agent (A2A) --- một tiêu chuẩn mở cho phép các agent từ nhiều nền tảng khác nhau giao tiếp với nhau qua mạng.

Mỗi agent có Agent Card mô tả khả năng

Tự động khám phá và tích hợp agent bên thứ ba

Phù hợp cho hệ sinh thái agent phân tán

```python

from strands.multiagent.a2a import A2AServer

a2a_agent = A2AServer(agent=local_agent, port=9000)

a2a_agent.serve()
```

Sẵn sàng cho môi trường production

Mặc dù đã được sử dụng nội bộ trong các dịch vụ như Amazon Q Developer và AWS Glue, Strands 1.0 được hoàn thiện dựa trên phản hồi từ hàng trăm khách hàng toàn cầu.

Quản lý session bền vững

SessionManager cho phép lưu trữ và khôi phục trạng thái agent từ các backend như:

File system

Amazon S3

Điều này đảm bảo agent duy trì đầy đủ ngữ cảnh ngay cả khi restart hoặc scale hệ thống.

Hỗ trợ bất đồng bộ và streaming

Strands 1.0 hỗ trợ async native trên toàn bộ stack, cho phép:

Thực thi song song nhiều agent

Streaming phản hồi theo thời gian thực

Hủy tác vụ khi người dùng rời khỏi giao diện

```python

async for event in agent.stream_async(message):

    ...
```

Hỗ trợ đa dạng mô hình AI

Strands cho phép sử dụng nhiều mô hình khác nhau cho từng agent, bao gồm:

Amazon Bedrock

OpenAI

Anthropic

Meta

Cohere

Mistral

Stability

Writer

Việc hoán đổi mô hình không yêu cầu thay đổi logic hoặc công cụ.

Kết luận

Strands Agents 1.0 đưa kiến trúc multi-agent từ giai đoạn thử nghiệm lên production-ready. Với khả năng mở rộng linh hoạt, hỗ trợ tiêu chuẩn mở như A2A và cộng đồng đóng góp mạnh mẽ, Strands đang trở thành một trong những nền tảng đơn giản và hiệu quả nhất để xây dựng hệ thống agent AI hiện đại.

Bắt đầu ngay hôm nay tại https://strandsagents.com