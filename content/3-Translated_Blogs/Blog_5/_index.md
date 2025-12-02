+++
title = "Blog 5"
weight =  5
chapter = false
pre = " <b> 3.5. </b>"
+++

# **Ra Mắt Strands Agents 1.0: Việc điều phối Multi-Agent cho môi trường production đã được đơn giản hóa**

_Ryan Coleman và Belle Guttman_ | 15/07/2025 | [Amazon Machine Learning](https://aws.amazon.com/blogs/opensource/category/artificial-intelligence/amazon-machine-learning/), [Announcements](https://aws.amazon.com/blogs/opensource/category/post-types/announcements/), [Artificial Intelligence](https://aws.amazon.com/blogs/opensource/category/artificial-intelligence/), [Open Source](https://aws.amazon.com/blogs/opensource/category/open-source/)| [Permalink](https://aws.amazon.com/blogs/opensource/introducing-strands-agents-1-0-production-ready-multi-agent-orchestration-made-simple/) | [Comments](https://aws.amazon.com/blogs/opensource/introducing-strands-agents-1-0-production-ready-multi-agent-orchestration-made-simple/#Comments)

Hôm nay, chúng tôi vui mừng thông báo về phiên bản 1.0 của [Strands Agents SDK](https://github.com/strands-agents/sdk-python), đánh dấu một cột mốc quan trọng trong hành trình giúp việc xây dựng các agent AI trở nên đơn giản, đáng tin cậy và sẵn sàng cho môi trường production. Strands Agents là một SDK mã nguồn mở, áp dụng phương pháp model-driven, giúp bạn xây dựng và vận hành các agent AI chỉ trong vài dòng code. Strands có khả năng mở rộng từ các trường hợp sử dụng agent đơn giản đến phức tạp, cũng như từ phát triển cục bộ đến triển khai trong môi trường production.

Kể từ khi ra mắt bản xem trước vào tháng 5 năm 2025, chúng tôi đã nhận được hơn 2.000 lượt sao trên GitHub và hơn 150 ngàn lượt tải xuống trên PyPI. Strands 1.0 mang đến mức độ đơn giản tương tự cho các ứng dụng multi-agent như những gì Strands đã làm được với các agent đơn lẻ, với việc bổ sung bốn primitive mới và hỗ trợ cho giao thức Agent to Agent (A2A). Để đưa kiến trúc multi-agent vào môi trường production, phiên bản 1.0 cũng bao gồm một trình quản lý session mới để truy xuất trạng thái agent từ một kho dữ liệu từ xa, cùng với việc cải thiện hỗ trợ bất đồng bộ xuyên suốt toàn bộ SDK. Nhằm tăng tính linh hoạt để xây dựng agent của bạn với bất kỳ mô hình nào, cStrands 1.0 đã mở rộng hỗ trợ thêm API của năm nhà cung cấp mô hình mới, được đóng góp bởi các đối tác như [Anthropic](https://www.anthropic.com/), [Meta](https://www.llama.com/), [OpenAI](https://openai.com/), [Cohere](https://cohere.com/), [Mistral](https://mistral.ai/), [Stability](https://stability.ai/), [Writer](https://writer.com/) và [Baseten](https://www.baseten.co/) (xem [pull request](https://github.com/strands-agents/sdk-python/pull/389)). Bây giờ, hãy cùng đi sâu vào chi tiết các cập nhật này. Các mẫu code đầy đủ có sẵn tại trang [strandsagents.com.](https://strandsagents.com/)

## Đơn giản hóa các mô hình multi-agent

Các mô hình multi-agent cho phép các agent AI chuyên biệt cùng nhau làm việc—phân công nhiệm vụ, chia sẻ kiến thức và phối hợp hành động—nhằm giải quyết những vấn đề phức tạp mà một agent đơn lẻ không thể xử lý được. Strands 1.0 giới thiệu bốn primitive trực quan, giúp việc điều phối nhiều agent trở thành một phần mở rộng đơn giản của tổ hợp mô hình/công cụ/prompt mà bạn vốn đã sử dụng để tạo ra các agent đơn lẻ.

**1. Agents-as-Tools: Đơn giản hóa việc ủy quyền theo cấp bậc**

Mô hình agents-as-tools biến các agent chuyên biệt thành những công cụ thông minh mà các agent khác có thể gọi đến. Điều này tạo điều kiện cho việc ủy quyền theo cấp bậc, nơi các agent hoạt động như người điều phối có thể chủ động tham vấn các chuyên gia theo từng lĩnh vực cụ thể mà không từ bỏ quyền kiểm soát yêu cầu. Điều này phản ánh cách thức làm việc của các đội nhóm con người — một quản lý dự án không cần phải biết mọi thứ, họ chỉ cần biết nên tham khảo ý kiến của chuyên gia nào cho từng nhiệm vụ cụ thể.

``` Python

from strands import Agent, tool 
from strands_tools import calculator, file_write, python_repl, journal 

@tool 
def web_search(query: str) -> str: 
    return "Dummy web search results here!" 

# Create specialized agents 
research_analyst_agent = Agent( 
    system_prompt="You are a research specialist who gathers and analyzes information about local startup markets", 
    tools=[web_search, calculator, file_write, python_repl] 
) 

travel_advisor_agent = Agent( 
    system_prompt="You are a travel expert who helps with trip planning and destination advice", 
    tools=[web_search, journal] 
) 

# Convert the agents into tools 
@tool 
def research_analyst(query: str) -> str: 
    response = research_analyst_agent(query) 
    return str(response) 

@tool 
def travel_advisor(query: str) -> str: 
    response = travel_advisor_agent(query) 
    return str(response) 

# Orchestrator naturally delegates to specialists 
executive_assistant = Agent(  
    tools=[research_analyst, travel_advisor]  
) 

result = executive_assistant("I have a business meeting in Portland next week. Suggest a nice place to stay near the local startup scene, and suggest a few startups to visit") 

```
Trong ví dụ rút gọn này, chúng ta định nghĩa hai agent du lịch và nghiên cứu, mỗi agent có prompt và công cụ chuyên biệt cho lĩnh vực của mình, mà agent trợ lý điều hành có thể gọi đến để lấy thông tin phục vụ yêu cầu của người dùng. agent trợ lý điều hành chịu trách nhiệm tổng hợp đầu vào từ các agent khác và tạo ra phản hồi gửi lại cho người dùng. Tìm hiểu thêm về mô hình [Agents-as-Tools](https://strandsagents.com/latest/user-guide/concepts/multi-agent/agents-as-tools/) trong tài liệu chính thức của Strands.

**2. Handoffs: chuyển giao quyền kiểm soát rõ ràng**

Tính năng Handoffs cho phép các agent chủ động chuyển trách nhiệm sang con người khi gặp phải nhiệm vụ vượt quá phạm vi chuyên môn của mình, đồng thời giữ nguyên toàn bộ ngữ cảnh cuộc trò chuyện trong quá trình chuyển giao. Strands cung cấp sẵn công cụ được tích hợp sẵn `handoff_to_user` giúp cho các agent có thể dùng để chuyển giao quyền kiểm soát một cách liền mạch trong khi vẫn duy trì lịch sử và ngữ cảnh cuộc trò chuyện - giống như một nhân viên dịch vụ khách hàng cung cấp chủ động yêu cầu khách cung cấp thêm thông tin về trường hợp của họ.

``` Python

from strands import Agent
from strands_tools import handoff_to_user

SYSTEM_PROMPT="""
Answer the user's support query. Ask them questions with the handoff_to_user tool when you need more information
"""

# Include the handoff_to_user tool in our agent's tool list
agent = Agent(
    system_prompt=SYSTEM_PROMPT,
    tools=[handoff_to_user]
)

# The agent calls the handoff_to_user tool which includes the question for the customer
agent("I have a question about my order.")

```

Các agent cũng có thể đặt câu hỏi cho con người khi được yêu cầu làm như vậy

``` Python

from strands import Agent

SYSTEM_PROMPT="""
Answer the user's support query. Ask them questions when you need more information
"""

agent = Agent(
    system_prompt=SYSTEM_PROMPT,
)

# The agent asks questions by streaming them back as text
agent("I have a question about my order.")


```

**3. Swarms: Các nhóm cộng tác tự tổ chức**

Một Swarm tạo ra các agent tự chủ phối hợp động thông qua việc chia sẻ bộ nhớ,cho phép nhiều chuyên gia có thể cộng tác cho các tác vụ phức tạp. Hãy hình dung nó giống như một buổi thảo luận nhóm, nơi các chuyên gia xây dựng ý tưởng dựa trên ý tưởng của nhau, với đội ngũ tự tổ chức để mang lại kết quả tập thể tốt nhất.

``` Python

import logging  
from strands import Agent 
from strands.multiagent import Swarm 
from strands_tools import memory, calculator, file_write 

# Enables Strands debug logs level, and prints to stderr 
logging.getLogger("strands.multiagent").setLevel(logging.DEBUG) 
logging.basicConfig( 
    format="%(levelname)s | %(name)s | %(message)s", 
    handlers=[logging.StreamHandler()] 
) 

researcher = Agent( 
    name="researcher", 
    system_prompt="You research topics thoroughly using your memory and built-in knowledge", 
    tools=[memory] 
) 

analyst = Agent( 
    name="analyst", 
    system_prompt="You analyze data and create insights", 
    tools=[calculator, memory] 
) 

writer = Agent( 
    name="writer", 
    system_prompt="You write comprehensive reports based on research and analysis", 
    tools=[file_write, memory] 
) 

# Swarm automatically coordinates agents 
market_research_team = Swarm([researcher, analyst, writer]) 

result = market_research_team( 
    "What is the history of AI since 1950? Create a comprehensive report" 
) 

```

Nghiên cứu thêm về [Swarms](https://strandsagents.com/latest/user-guide/concepts/multi-agent/swarm/) trong tài liệu về Strands

**4. Graphs: Kiểm soát quy trình làm việc mang tính xác định**

Graphs cho phép bạn xác định quy trình làm việc của các agent một cách rõ ràng với những định tuyến có điều kiện và những điểm ra quyết định, rất hữu ích cho những quy trình yêu cầu các bước cụ thể, cơ chế phê duyệt hoặc các ngưỡng kiểm soát chất lượng. Giống như một dây chuyền lắp ráp hoặc chuỗi phê duyệt được thiết kế tốt, graphs đảm bảo các agent luôn tuân thủ các quy tắc nghiệp vụ đã định sẵn theo đúng trình tự mỗi lần thực thi.

``` Python
from strands import Agent  
from strands.multiagent import GraphBuilder  

analyzer_agent = Agent(  
    name="analyzer",  
    system_prompt="Analyze customer requests and categorize them",  
    tools=[text_classifier, sentiment_analyzer]  
) 

normal_processor = Agent(  
    name="normal_processor",  
    system_prompt="Handle routine requests automatically",  
    tools=[knowledge_base, auto_responder]  
)  

critical_processor = Agent(  
    name="critical_processor",  
    system_prompt="Handle critical requests quickly",  
    tools=[knowledge_base, escalate_to_support_agent]  
)  

# Build deterministic workflow  
builder = GraphBuilder()  
builder.add_node(analyzer_agent, "analyze")  
builder.add_node(normal_processor, "normal_processor")  
builder.add_node(critical_processor, "critical_processor") 

# Define conditional routing 
def is_approved(state): 
    return True 

def is_critical(state): 
    return False 

builder.add_edge("analyze", "normal_processor", condition=is_approved)  
builder.add_edge("analyze", "critical_processor", condition=is_critical)  
builder.set_entry_point("analyze")  
customer_support_graph = builder.build() 

# Execute the graph with user input 
results = customer_support_graph("I need help with my order!") 

```

Nghiên cứu thêm về [Graphs](https://strandsagents.com/latest/user-guide/concepts/multi-agent/graph/) trong tài liệu về Strands

Các mô hình multi-agent (multi-agent patterns) này được thiết kế để được áp dụng dần dần và kết hợp một cách tự do — hãy bắt đầu với các agent đơn lẻ, thêm các chuyên gia như là công cụ, phát triển thành các Swarms, và điều phối bằng các Graphs khi nhu cầu của bạn tăng lên. Hãy kết hợp và tùy chỉnh các mô hình để tạo ra các hệ thống tinh vi: swarms có thể chứa các graphs, graphs có thể điều phối các swarms, và bất kỳ mô hình nào cũng có thể sử dụng các agent được trang bị thêm các agent khác làm công cụ.

``` Python
from strands import Agent, tool 
from strands.multiagent import GraphBuilder, Swarm 
from strands_tools import memory, calculator, python_repl, file_write  

# Start simple with a single agent 
agent = Agent(tools=[memory]) 

# Create specialist agents that a lead orchestrator agent can consult 
data_analyst = Agent(name="analyst", tools=[calculator, python_repl]) 

@tool 
def data_analyst_tool(query: str) -> str: 
    return str(data_analyst(query)) 

analyst_orchestrator = Agent(tools=[memory, data_analyst_tool]) # Agents-as-tools 

# Compose patterns together - a graph that uses a swarm 
researcher = Agent(name="researcher", tools=[memory]) 
writer = Agent(name="writer", tools=[file_write]) 
research_swarm = Swarm([researcher, analyst_orchestrator, writer]) 
review_agent = Agent(system_prompt="Review the research quality and suggest improvements") 
builder = GraphBuilder() 
builder.add_node(research_swarm, "research") # Swarm as graph node 
builder.add_node(review_agent, "review") 
builder.add_edge("research", "review") 
graph = builder.build() 

# The patterns nest naturally - swarms in graphs, agents as tools everywhere  
result = graph("How has green energy evolved over the last few years?") 
```
## Hệ thống Multi-Agent với A2A

Strand 1.0 đã bao gồm việc hỗ trợ cho [giao thức Agent-to-Agent (A2A)](https://a2aproject.github.io/A2A/latest/), một tiêu chuẩn mở cho phép các agent từ các nền tảng khác nhau có thể giao tiếp một cách liền mạch. Bất kỳ agent Strands cũng có thể được tích hợp các khả năng A2A để trở nên có thể truy cập qua mạng và tuân thủ giao thức A2A. Các agent A2A từ các tổ chức bên ngoài cũng có thể được sử dụng trực tiếp trong tất cả các mô hình multi-agent của Strands.

``` Python
from strands import Agent 
from strands.multiagent.a2a import A2AServer 
from strands_tools.a2a_client import A2AClientToolProvider 

# Serve your agent via A2A protocol 
local_agent = Agent(name="analyzer", tools=[web_search, data_analysis]) 
a2a_agent = A2AServer(agent=local_agent, port=9000) 
a2a_agent.serve() # AgentCard available at http://localhost:9000/.well-known/agent.json 

# Use remote A2A agents 
partner_agent_url = "https://partner.com" 
cloud_agent_url = "https://cloud.ai" 

# Connect to remote A2A enabled agents 
a2a_tool_provider = A2AClientToolProvider(known_agent_urls=[partner_agent_url, cloud_agent_url]) 

# Orchestrate remote agents 
orchestrator = Agent(tools=[a2a_tool_provider.tools]) 
```

Bởi vì A2A cung cấp các tính năng như agent card, một mô tả được chuẩn hóa về khả năng của agent, các hệ thống multi-agent được bật A2A có thể dễ dàng khám phá và kết nối với các agent được tạo ra bởi các nhóm hoặc các tổ chức khác. Strands tự động tạo ra thẻ agent dựa trên các công cụ bạn đã cung cấp cho agent. Để xem các ví dụ hoạt động hoàn chỉnh và bắt đầu với tính năng tích hợp A2A, hãy tham khảo [repository mẫu](https://github.com/strands-agents/samples/tree/main/03-integrations/Native-A2A-Support) của chúng tôi và [tài liệu A2A của Strands](https://strandsagents.com/latest/user-guide/concepts/multi-agent/agent-to-agent/).

## Sẵn sàng cho môi trường production

Mặc dù Strands đã được các đội nội bộ của Amazon như Amazon Q Developer và AWS Glue sử dụng trong môi trường production từ lâu trước khi ra mắt công chúng, chúng tôi đã và đang làm việc ngược lại với hàng trăm khách hàng trên toàn thế giới để mở rộng Strands nhằm đáp ứng các nhu cầu production của bạn. Các cập nhật này bao gồm một tầng trừu tượng quản lý session để hỗ trợ việc lưu trữ liên tục dữ liệu vào và phục hồi từ các kho dữ liệu bên ngoài, đầu ra có cấu trúc, cải thiện hỗ trợ bất đồng bộ, và nhiều thứ khác nữa (xem chi tiết trong [changelog các phiên bản phát hành](https://github.com/strands-agents/sdk-python/releases))

**Quản lý session bền vững:** Chúng tôi đã thêm vào `SessionManager`, một công cụ trừu tượng hóa quản lý session cho phép tự động lưu trữ liên tục và khôi phục lịch sử hội thoại cũng như trạng thái của agent. Nhờ đó, các agent có thể lưu toàn bộ lịch sử cuộc trò chuyện vào một hệ thống lưu trữ như Amazon Simple Storage Service (Amazon S3) và tiếp tục cuộc hội thoại một cách liền mạch ngay cả sau khi hệ thống khởi động lại. Dưới đây là một ví dụ sử dụng cơ chế lưu trữ liên tục dựa trên file cơ bản.

``` Python
from strands import Agent 
from strands.session.file_session_manager import FileSessionManager 

# Create a session manager with file-based storage 
Session_manager = FileSessionManager(session_id=”customer_support”, base_dir="./agent_sessions") 

# Agent automatically persists all conversations 
agent = Agent( 
    id="support_bot_1", 
    session_manager=session_manager, 
    tools=[knowledge_base, ticket_system] 
) 

# Messages are automatically saved as the conversation progresses 
agent("Help me reset my password") 
agent("I can't access my email") 

# Later, even after a restart, restore the full conversation 
```

Bạn có thể mở rộng lớp trừu tượng này bằng cách tự triển khai backend lưu trữ của riêng mình thông qua mẫu thiết kế Data Access Object (DAO), và Strands đã tích hợp sẵn hai backend mặc định: hệ thống file cục bộ và Amazon S3. Mỗi agent nhận một ID duy nhất để theo dõi, và hệ thống xử lý các agent đồng thời  trong cùng một session cho các kịch bản multi-agent, đảm bảo rằng các agent luôn duy trì ngữ cảnh qua các lần triển khai, mở rộng quy mô hệ thống hoặc khởi động lại. Tìm hiểu thêm về [Session Management](https://strandsagents.com/latest/user-guide/concepts/agents/session-management/) trong tài liệu của Strands.

**Hỗ trợ bất đồng bộ nguyên bản và cải thiện hiệu năng:** Các workload trong môi trường production đòi hỏi độ tin cậy cao và hiệu năng phản hồi nhanh. Trong phiên bản 1.0, chúng tôi đã cải tiến kiến trúc vòng lặp sự kiện của Strands để hỗ trợ thao tác bất đồng bộ trên toàn bộ stack. Các công cụ và nhà cung cấp mô hình giờ đây có thể chạy bất đồng bộ mà không bị chặn, cho phép thực thi đồng thời thực sự. Phương thức `stream_async` mới truyền phát tất cả các sự kiện của agent — văn bản, việc sử dụng công cụ, các bước suy luận — theo thời gian thực, vđồng thời tích hợp cơ chế hủy khi người dùng rời khỏi trang.

``` Python
import asyncio 
from fastapi import FastAPI 
from fastapi.responses import StreamingResponse 
from strands import Agent 
from strands_tools import calculator 

app = FastAPI() 
@app.post("/chat") 

async def chat_endpoint(message: str): 
    async def stream_response(): 
        agent = Agent(tools=[web_search, calculator]) 
        # Stream agent responses in real-time 
        async for event in agent.stream_async(message): 
            if "data" in event: 
                yield f"data: {event['data']}\n\n" 
            elif "current_tool_use" in event: 
                yield f"event: tool\ndata: Using {event['current_tool_use']['name']}\n\n" 
    return StreamingResponse(stream_response(), media_type="text/event-stream") 

# Concurrent agent evaluation 
async def evaluate_models_concurrently(prompt: str): 
    async def stream(agent: Agent): 
        print(f"STARTING: {agent.name}") 
        async for event in agent.stream_async(prompt): 
            # handle events 
        print(f"ENDING: {agent.name}") 
        return event[“result”]  # last event is the agent result 

    agents = [ 
        Agent(name="claude", model="us.anthropic.claude-3-7-sonnet-20250219-v1:0”), 
        Agent(name="deepseek”, model="us.deepseek.r1-v1:0”), 
        Agent(name="nova", model="us.amazon.nova-pro-v1:0") 
    ] 

    # Execute all agents concurrently 
    responses = await asyncio.gather(*[stream(agent) for agent in agents]) 

    return responses 
```

Tìm hiểu thêm về [Hỗ trợ bất động bộ nguyên bản](https://strandsagents.com/latest/user-guide/concepts/streaming/async-iterators/) trong tài liệu chính thức của Strands.

**Mở rộng hỗ trợ đa dạng nhà cung cấp mô hình:** Khách hàng đã chia sẻ rằng họ cần linh hoạt trong việc sử dụng các mô hình khác nhau cho các tác vụ khác nhau. Để đáp ứng nhu cầu này, Strands Agents đã nhận được sự hỗ trợ mạnh mẽ từ cộng đồng các nhà cung cấp mô hình. Các nhà cung cấp như [Anthropic](https://www.anthropic.com/), [Meta](https://www.llama.com/), [OpenAI](https://openai.com/), [Cohere](https://cohere.com/), [Mistral](https://mistral.ai/), [Stability](https://stability.ai/) và [Writer](https://writer.com/) đã đóng góp cho phép API mô hình của họ được sử dụng bởi một Strands Agent thông qua code. Việc truy cập Strands Agents thông qua hạ tầng API do chính các nhà cung cấp này cung cấp giúp các nhà phát triển tập trung vào việc xây dựng các giải pháp AI-powered, mà không cần lo lắng về quản lý cơ sở hạ tầng. Những bổ sung này bổ trợ hoàn hảo cho khả năng hỗ trợ từ giai đoạn xem trước đối với mọi mô hình trên Amazon Bedrock, OpenAI, và bất kỳ endpoint tương thích OpenAI nào thông qua LiteLLM. Strands cho phép bạn sử dụng các mô hình khác nhau cho mỗi agent, hoặc chuyển đổi mô hình và nhà cung cấp mô hình mà không cần sửa đổi công cụ hoặc logic của bạn.

``` Python
from strands import Agent
from strands.models import BedrockModel
from strands.models.openai import OpenAIModel
from strands.models.anthropic import AnthropicModel

# Configure different model providers
bedrock_model = BedrockModel(
    model_id="us.amazon.nova-pro-v1:0",
    temperature=0.3,
    top_p=0.8,
    region_name="us-west-2"
)

openai_model = OpenAIModel(
    client_args={
        "api_key": "your-api-key",
    },
    model_id="gpt-4o",
    params={
        "max_tokens": 1000,
        "temperature": 0.7,
    }
)

anthropic_model = AnthropicModel(
    client_args={
        "api_key": "your-api-key",
    },
    max_tokens=1028,
    model_id="claude-3-7-sonnet-20250219",
    params={
        "temperature": 0.5,
    }
)

# Swap models or use different models for different agents in the same system
researcher = Agent(
    name="researcher",
    model=anthropic_model,
    tools=[web_search]
)

writer = Agent(
    name="writer", 
    model=openai_model,
    tools=[document_formatter]
)

analyzer = Agent(
    name="analyzer",
    model=bedrock_model,
    tools=[data_processor]
)
```

Cộng đồng Strands đã đóng vai trò then chốt trong việc định hình tất cả những cải tiến này thông qua việc sử dụng thực tế, phản hồi và những đóng góp mã nguồn trực tiếp. Trong số hơn 150 pull request (PR) đã được merge vào Strands từ phiên bản 0.1.0 đến 1.0, 22% được đóng góp bởi các thành viên cộng đồng, những người đã sửa lỗi, thêm nhà cung cấp mô hình, viết tài liệu, thêm tính năng và tái cấu trúc các lớp để cải thiện hiệu suất. Chúng tôi vô cùng biết ơn [mỗi người trong số các bạn](https://github.com/strands-agents/sdk-python/graphs/contributors) vì đã giúp Strands trở thành cách đơn giản nhất để đưa một agent từ nguyên mẫu đến triển khai thực tế.

Tương lai của AI là multi-agent (multi-agent), và với Strands 1.0, tương lai đó đã sẵn sàng để triển khai trong môi trường production. Hãy bắt đầu xây dựng ngay hôm nay tại [strandsagents.com.](https://strandsagents.com/)