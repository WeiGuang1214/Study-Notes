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