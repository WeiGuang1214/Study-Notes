## SpringAI和LangChain4j引入

​	为什么需要SpringAI和LangChain4j？首先通过传统的方式也可以调用大模型，比如HTTP client或者是 OK http，通过get请求可以拼接前端传入的query进行交互，然后接收大模型的流式响应结果，再通过BufferedReader一行行读，可以处理JSON结果，这是不需要AI框架情况下构建RAG工程的方法，一般写在服务端后台，用springboot去做。

​	但是如果想做一个AI应用，就需要引入框架了，SpringAI和LangChain4j是目前主流的服务端AI架构，。

​	SpringAI和LangChain4j封装了上层API，优先Spring AI Alibaba和Spring AI

​	需要注入依赖并且设置环境变量，api url和api key，才可以调用对应model

```java
@Test
void test(){
	ChatLanguageModel model = QwenChatModel
		.builder()
		//.baseUrl("https://dashscope.aliyuncs.com")
		.apiKey(System.getenv(name:"ALI_AI_KEY"))
		.modelName("qwen-max")
		.build();
	String answer=model.chat(userMessage:"你好，你是谁?");
	System.out.println(answer);
}
```

相应有文生图、文生语音，阿里官网有demo。

### LangChain4j整合SpringBoot

参考官网，主要使用SpringAI完成。

### SpringAI

#### 一、核心概念

##### 1、模型(Model):各种模态的LLM。

##### 2、提示(Prompt):Prompt 作为语言基础输入的基础，指导 A1 模型生成特定的输出。

##### 3、提示词模板(Prompt Template):

​	涉及建立请求的上下文，并用用户输入的特定值替换请求的部分内容，例如：

```java
Tell me a fadjective} joke about {content}.
```

##### 4、嵌入(Embedding)

​	Embedding 通过将文本、图像和视频转换为称为向量(Vector)的浮点数数组来工作。这些向量旨在捕捉文本、图像和视频的含义，Embedding 数组的长度称为向量的维度。
​	通过计算两个文本片段的向量表示之间的数值距离，可以确定用于生成嵌入向量的对象之间的相似性。

##### 5、Token:

​	语料，输入时，模型将单词转换为 token。输出时，它们将 token 转换回单词。

##### 6、结构化输出(Structured Output):

​	要求回复为 JSON ，AI模型的输出通常也会以java.lang.String 的形式出现。它可能是正确的JSON，但它可能并不是你想要的 JSON 数据结构，它只是一个字符串。此外，在提示词 Prompt 中要求“返回 JSON”并非 100% 准确。

##### 7、如何将数据和 API 引入 AI 模型?

1、Fine Tuning 微调：
2、Prompt Stuffing 提示词填充：

​	一种更实用的替代方案是将您的数据嵌入到提供给模型的提示中

​	Spring Al 库可帮助您基于“提示词填充”技术，也称为检索增强生成 (RAG)实现解决方案。需要构建矢量数据库，并且拆分文档(适配大模型输入上下文长度)。

4、Function Calling：LLM在训练后即被冻结，导致知识陈旧，并且无法访问或修改外部数据

##### 	本质上是大模型感知了用户输入中的参数和函数名，通过API去调用该方法，并且把结果处理返回给用户，可以通过function call调用服务端的代码，为用户提供服务端的业务功能。

##### 8、评估人工智能的回答：ragas或者人工打分

### Demo:

快速开始，下载springai.alibaba.example

```java
git clone --depth=1 https://github.com/springaialibaba/spring-ai-alibaba-examples git
cd spring-ai-alibaba-examples/spring-ai-alibaba-helloworld
```

1、添加spring-ai-alibaba-starter依赖

```java
<dependency>
<groupId>com.alibaba.cloud.ai</qroupId>
<artifactId>spring-ai-alibaba-starter</artifactId>
<version>1.0.0-M5.1</version>
</dependency>
```

##### 2、注入 ChatClient并且设置默认Prompt（理解为一种前提设定）

​	SpringAl提供了Chatclient来和大模型进行交互，最简单的一种，直接输入query进行交互

```java
@RestController
@RequestMapping("/helloworld")
public class HelloworldController {
  private static final String DEFAULT_PROMPT = "你是一个博学的智能聊天助手，请根据用户提问回答！";

  private final ChatClient dashScopeChatClient;

  public HelloworldController(ChatClient.Builder chatClientBuilder) {
    this.dashScopeChatClient = chatClientBuilder
        .defaultSystem(DEFAULT_PROMPT)
         // 实现 Chat Memory 的 Advisor
         // 在使用 Chat Memory 时，需要指定对话 ID，以便 Spring AI 处理上下文。
         .defaultAdvisors(
             new MessageChatMemoryAdvisor(new InMemoryChatMemory())
         )
         // 实现 Logger 的 Advisor
         .defaultAdvisors(
             new SimpleLoggerAdvisor()
         )
         // 设置 ChatClient 中 ChatModel 的 Options 参数
         .defaultOptions(
             DashScopeChatOptions.builder()
                 .withTopP(0.7)
                 .build()
         )
         .build();
   }

  @GetMapping("/simple/chat")
  public String simpleChat(String query) {
    return dashScopeChatClient.prompt(query).call().content();
  }
 }
```

图1

核心功能:
模型交互-ChatClient
ChatClient 类似于服务层，它为应用程序直接提供 AI服务，提供多种工具。

- 定制和组装模型的输入(Prompt)

- 格式化解析模型的输出(Structured Output)

- 调整模型交互参数(ChatOptions)

- ##### 工具/函数调用(Function Calling) -- 核心 -- 商品手卡

- ##### RAG -- 主播知识库

##### 1、Chatclient:

例 如 : 设 置 input,call 方 法 向 AI 发 送 请 求 ,content 方 法 以 字 符 串 形 式 返 回 AI 模 型 的 响 应

```java
	@RestController
	public class ChatController {
		private final ChatClient chatClient;
		public ChatController(ChatClient.Builder builder) {
			this.chatClient = builder.build();
        }
	@GetMapping("/chat")
	public String chat(String input) {
		return this.chatClient.prompt().user(input).call ().content ();
    }
}
```

​	可以在ChatClient中禁用默认的bean自动配置，选择自己的chatmodel，重新指定chatmodel
并且生成builder，自定义build()；

```java
ChatModel myChatModel = ... // usually autowired
ChatClient.Builder builder = ChatClient.builder(myChatModel)
// or create a ChatClient with the default builder settings:
ChatClient chatClient = ChatClient.create(myChatModel);
```

##### call() -同步执行请求

##### stream() -流式执行请求

格式化AI响应的结果：
	返回ChatResponse：包含响应生成相关的元数据,同时它还可以包含多个子响应(称为
Generation)
	返回实体类(Entity)：从String到实体类的转换,

```java
record ActorFilms(String actor, List<String> movies) {
	ActorFilms actorFilms = chatClient.prompt()
						.user("Generate the filmography for a random actor.")
						.call()
						.entity(ActorFilms.class);
}
```

指定泛型:

```java
List<ActorFilms> actorFilms = chatClient.prompt()
		.user("Generate the filmography of 5 movies for Tom Hanks and
Bill Murray.")
		.call()
		.entity(new ParameterizedTypeReference<List<ActorFilms>>() {
		});
```

流式响应：

```java
Flux<String> output = chatClient.prompt()
						.user("Tell me a joke")
						.stream()
						.content () ;
```

Flux<String>content():返回由AI模型生成的字符串的Flux。
Flux<ChatResponse>chatResponse():返回对象的Flux ChatResponse,其中包含有关响应的附加
元数据。

##### 定制 ChatClient默认值

1、设置默认的聊天前提：
	在创建chatclient实例时：

​	builder.defaultSystem("You are a friendly chat bot that answersquestion in the voice of a Pirate").build();
​	也可以是用动态参数配置systemPrompt,类似于prompTemplate；

```java
	builder.defaultSystem("Youare a friendly chat bot that answers question in the voice of a {voice}").build();
	// 然后 
	chatClient.prompt()
			.system(sp -> sp.param("voice", voice))
			.user(message)
			.call()
			.content();
```



##### 检索增强生成(RAG)：

​	向量数据库存储的是Al模型不知道的数据，当用户问题被发送到Al模型时，QuestionAnswerAdvisor 会在向量数据库中查询与用户问题相关的文档。来自向量数据库的响应被附加到用户消息Prompt中,为Al模型生成响应提供上下文。

```java
ChatResponse response = ChatClient.builder(chatModel)
									.build()
        							.prompt()
						.advisors(new QuestionAnswerAdvisor(vectorStore,SearchRequest.defaults()))
   					 	.user(userText)
						.call()
						.chatResponse();
```

日志记录：启用日志记录，请在创建ChatClient 时将SimpleLoggerAdvisor 添加到Advisor链中。建
议将其添加到链的末尾：

```java
ChatResponse response = ChatClient.create(chatModel).prompt()
								.advisors(new SimpleLoggerAdvisor())
				.user("Tell me a joke?")
				.call(
				.chatResponse();
```

将Advisor包的日志记录级别设置为DEBUG:org.springframework.ai.chat.client.advisor=DEBUG
添加到application.yaml中
自定义数据格式：

```java
SimpleLoggerAdvisor(
	Function<AdvisedRequest, String> requestToString,
	Function<ChatResponse, String> responseToString;
)
```

2. ##### SpringAl中的prompt：明确 AI模型提示的每个部分的上下文和目的

```java
{"messages":
	[
		{"role":"user","content":"<用户输入1>"},
		{"role":"systemRole","content":"<模型输出1>“},//向AI提供前提设定
		{"role":"assistant","content":"<模型输出1>"},
		{"role":"tool/FunctionRole","content":"<模型输出1>"}
	]
}
```

##### Prompt Template

```java
@RestController
@RequestMapping("/example/aidemo")
public class DemoController {
	private final ChatClient chatClient;
	public DemoController(ChatClient.Builder builder) {
		this.chatClient = builder.build();
	}
	@GetMapping("/chat")
	public Flux<String> generate(
		@RequestParam(
			value = "message",
			required = false,
			defaultValue = "Tell me about three famous pirates from the Golden Age of Piracy and why they did. Write at least a sentence for each pirate.") String message,

		@RequestParam(value = "name", required = false,defaultValue = "Bob") String name,
		@RequestParam(value = "voice", required = falsp,defaultValue = "pirate") String voice){
        // 用户输入
		UserMessage userMessage = new UserMessage (message);
		String systemText = "You are a helpful AI assistant that helps people find information,Your name is {name},You should reply to the user's request with your name and also in the style of a {voice}.";
	SystemPromptTemplate systemPromptTemplate = new SystemPromptTemplate(systemText);
	// 使用 System prompt tmpl
	Message systemMessage =systemPromptTemplate.createMessage(Map.of("name", name, "voice"));
	return chatClient.prompt(
        	new Prompt(List.of(userMessage,systemMessage)))
        	.stream()
        	.content();
    }
```

