---
author: "Jerry Fang"
handle: "@Jerry_fsy"
source: "https://x.com/Jerry_fsy/status/2112844396316213433"
date: 2026-10-21T09:52:13.000Z
type: tweet
---
# Jerry Fang (@Jerry_fsy)

从0搭建一个Agent（1）：Agent Loop + Tool System  
  
在Sitor上跟着“Super Agent实战课”完成了前两章的学习（Agent Loop + Tool System）。在这里做一下总结，希望能帮刚入门 Agent 的朋友少踩一些坑。  
  
具体如何从0搭建一个Agent在这个系列中因为已经有Sitor上的“Super Agent实战课”的付费课程和Datawhale的hello-agents这样的开源学习资料，所以不会详细阐述，这个系列主要讲解在这个过程中需要注意的一些点。  
  
第一，在使用模型API前先做一个Mock Model。  
从0搭建一个Mock Model可以帮助我们更好的理解LanguageModelV3 接口，知道generateText 和 streamText是如何实现的以及它们之间的区别。  
当然，既然是Mock Model，模型的回复就做的简单一些。  
  
第二，在Agent Loop中需要加层层保护。  
Agent Loop 的核心是：模型思考，调工具，拿结果，再思考。  
但是如果不做好层层保护的话，在生产端的Agent Loop很难稳定运行。尤其是下面三种防护要格外注意：  
1.循环检测：我们不希望模型在不断做事，但没有任何进展，尤其是下面两种情况：  
（1）通用重复：同一个工具，同样的参数，同样的返回结果，然后反复调用；  
（2）乒乓循环：两个操作来回交替，A→B→A→B，看起来是在“做事”，但实际毫无进展。  
  
2.API容错：针对不同的API错误，采取不同的应对策略，而不是使用统一try-catch方案。关键是对API的报错做一个分类，对不同的类别采取对应的措施。  
  
3.Token预算：我们不希望随着Agent的狂奔，消耗的金额不受控制。因此，设定一个token的预算对于控制整体成本是一道保险。  
  
第三，模型和Agent Loop对于工具的关注角度是不同的。  
模型关注的是怎么调用这些工具，想知道name, description, parameters等参数；  
但是Agent Loop还需要知道工具A和其他工具能否并行执行，这些工具会不会改状态，结果会不会过长等等。这些模型不会关注，但是关乎到整个Agent的执行稳定性和效率，因此需要格外注意。  
  
第四，理解如果不做工具的延迟加载，为什么工具越多，模型检索到正确的工具的概率可能会下降。