# AI

### 1. LLM(Large Language Model)

- 대규모 텍스트 데이터를 학습하여 다음에 올 토큰을 예측하는 확률형 언어 모델
- 인간이 규칙을 하드코딩하는 기존 소프트웨어와 달리, 방대한 데이터의 패턴을 학습해 자연어를 이해하고 생성함.

### 2. Token

- LLM이 텍스트를 처리하는 최소 단위(문자, 단어 혹은 형태소)
- AI API 비용 체계의 기본 기준, 긴 대화 기록이나 대용량 문서 처리 시 토큰 최적화 필요

### 3. Context Window

- 모델이 한 번에 참고할 수 있는 입력 데이터 최대 범위.
- 대화 기록, 시스템 프롬프트, 사용자 질문이 모두 포함되며, 용량 초과시 과거 대화 누락 및 답변 품질 저하, 대기시간(Latency) 증가 발생

### 4. Hallucination

- 모델이 사실이 아닌 내용을 그럴듯하게 생성하는 현상
- 정답을 찾는 데이터베이스가 아니라 확률적 다음 토큰 예측 모델이기에 발생하며, 이를 해결하기 위해 RAG나 Function Calling을 연동

### 5. RAG(Retrieval Augmented Generation)

- 외부 데이터 검색결과와 LLM 생성 능력 결합한 아키텍처
- 학습 시점 이후의 최신 정보나 사내 보안 문서를 벡터 검색으로 먼저 찾아낸 뒤, 그 내용을 LLM에 프롬프트로 주입하여 답변의 정확도를 높임

### 6. Embedding

- 텍스트를 고차원의 숫자 배열(Vector)로 변환하여 컴퓨터가 의미를 이해하도록 하는 기술
- 의미가 유사한 문장이나 단어는 벡터 공간에서도 물리적으로 가까운 위치에 위치하게 됨

### 7. Vector Database

- 임베딩된 수많은 고차원 벡터 데이터 간의 유사도(거리)를 빠르게 계산하고 검색하기 위해 설계된 DB
- RAG 패턴에서 유저의 질문과 가장 연관성이 높은 문서 조각을 찾아낼 때 핵심 도구로 사용됨

### 8. Prompt Engineering

- 원하는 결과물의 품질을 높이기 위해 입력값을 구조화하여 설계하는 기술
- 개발 단계에서는 페르소나를 지정하는 System Prompt, 예시를 제공하는 Few-shot 등을 주로 활용함.

### 9. Function Calling

- LLM이 사용자 질문을 분석하여 "어떤 외부 API나 함수를, 어떤 인자값(Arguments)으로 호출해야 하는지 JSON 형태로 판단"해 주는 기능
- 실제 API 호출은 개발자의 애플리케이션 코드(백엔드/프론트엔드)에서 실행함.

### 10. AI Agent와 일반 챗봇 차이

- 일반 챗봇은 입력된 질문에 대한 일회성 답변을 생성하는 데 집중
- AI Agent는 스스로 목표를 설정하고, 계획을 수립하며,  필요한 도구를 선택해 실행하고 결과까지 검증하는 다단계 자율 수행 시스템

### 11. MCP(Model Context Protocol)

- AI 모델과 로컬 파일, DB, 외부 개발 도구 등을 안전하고 표준화된 방식으로 연결하는 오픈 프로토콜
- 모델마다 파편화되어 있던 도구 연동 방식을 하나로 통일해 AI 생태계 호환성 높임

### 12. 프론트엔드 개발자의 역할

- UX/UI 구현: 타이핑 효과를 위한 Streaming(SSE) 응답 처리, 유연한 채팅 UI 구축
    - LLM 답변 한번에 받으면 UI가 몇 초간 멈춰있는 것처럼 보이는데 이를 해결하기 위해 서버가 글자 조각(Chunk)를 보낼 때마다 프론트엔드가 실시간으로 화면을 업데이트
        
        ```tsx
        import { useState } from 'react';
        
        export default function ChatComponent() {
          const [input, setInput] = useState('');
          const [reply, setReply] = useState('');
        
          const handleSubmit = async (e: React.FormEvent) => {
            e.preventDefault();
            setReply('');
        
            const response = await fetch('/api/chat', {
              method: 'POST',
              body: JSON.stringify({ message: input }),
            });
        
            if (!response.body) return;
        
            // 1. 스트림 리더 생성
            const reader = response.body.getReader();
            const decoder = new TextDecoder();
            let done = false;
        
            // 2. 청크가 들어올 때마다 반복해서 읽기
            while (!done) {
              const { value, done: doneReading } = await reader.read();
              done = doneReading;
              
              const chunkValue = decoder.decode(value);
              // 3. 기존 답변 뒤에 실시간으로 글자 조각을 붙여줌 
              //   (유저는 타이핑 효과로 인지)
              setReply((prev) => prev + chunkValue);
            }
          };
        
          return (
            <div>
              <form onSubmit={handleSubmit}>
                <input value={input} onChange={(e) => setInput(e.target.value)} />
                <button type="submit">전송</button>
              </form>
              <div className="chat-box">{reply}</div>
            </div>
          );
        }
        ```
        
- 상태 및 데이터 관리: 무한 스크롤 기반의 대화 히스토리 관리, 토큰 절약을 위한 컨텍스트 요약 및 관리
- 인터랙션 인터페이스: Function Calling 결과를 유저가 보기 쉽게 차트, 모달 등의 UI로 시각화.
    - LLM은 직접 API를 보내지 않고 JSON 데이터만 던짐
        - 사용자가 날씨 물어보면
        - APP → LLM: 사용자가 날씨 물어봤어. 내가 날씨 API(get_weather) 가지고 있으니까 필요하면 양식에 맞춰서 요청해
        - LLM → APP: get_weather(location: “Seoul”) 형태로 실행해(JSON 리턴)
        - APP → 3rd Party → LLM: 앱이 직접 실제 날씨 API 호출해 결과 얻은 뒤, 그 결과값을 다시 LLM에 전달해 최종 답변 생성
        
        ```tsx
        // 1. 내가 가진 실제 자바스크립트 함수
        async function getWeather(location: string) {
          const res = await fetch(`https://weather-api.com{location}`);
          const data = await res.json();
          return JSON.stringify(data); // 예: { temp: "22도", condition: "맑음" }
        }
        
        async function handleAIChat(userMessage: string) {
          // 2. AI에게 우리가 가진 '도구(함수)' 스펙을 정의해서 함께 보냄
          const response = await openai.chat.completions.create({
            model: 'gpt-4o',
            messages: [{ role: 'user', content: userMessage }],
            tools: [{
              type: 'function',
              function: {
                name: 'getWeather',
                description: '특정 지역의 현재 날씨 정보를 가져옵니다.',
                parameters: {
                  type: 'object',
                  properties: { location: { type: 'string' } },
                  required: ['location'],
                },
              },
            }],
          });
        
          const message = response.choices[0].message;
        
          // 3. LLM이 "함수 실행이 필요하다"고 판단했는지 확인
          if (message.tool_calls) {
            const toolCall = message.tool_calls[0];
            
            if (toolCall.function.name === 'getWeather') {
              // LLM이 추출해 준 파라미터(Seoul) 파싱
              const args = JSON.parse(toolCall.function.arguments); 
              
              // 4. [핵심] 개발자가 작성한 실제 함수를 우리가 실행함!
              const weatherResult = await getWeather(args.location);
        
              // 5. 실행 결과를 다시 LLM에 보내서 자연스러운 문장으로 받아옴
              const finalResponse = await openai.chat.completions.create({
                model: 'gpt-4o',
                messages: [
                  { role: 'user', content: userMessage },
                  message, // LLM이 함수 호출을 요청했던 기록
                  { role: 'tool', tool_call_id: toolCall.id, content: weatherResult } // 함수 실행 결과
                ],
              });
        
              console.log(finalResponse.choices[0].message.content);
              // 출력: "현재 서울의 날씨는 22도로 아주 맑습니다!"
            }
          }
        }
        
        ```
        
- 비용 및 성능 최적화: 로딩 스피너 처리를 통한 인지 대기 시간 단축, 불필요한 API 호출 방지 UX 설계

### 13. 백엔드 개발자의 역할

- AI API 보안 및 프록시: 외부 AI 서비스의 API 키(Secret Key)를 노출하지 않고 안전하게 숨겨 관리
- RAG 파이프라인 구축: 사내 데이터(PDF, DB 등)를 쪼개고 가공하는 데이터 전처리(ETL)와 임베딩 후 Vector DB에 저장·검색하는 시스템 구현
- 대화 세션 및 메모리 관리: 자체 기억 능력이 없는 LLM을 보완하기 위해 Redis 나 RDBMS를 활용해 유저별 대화 이력을 관리하고, 토큰 절약을 위해 과거 대화를 요약·압축하는 로직 구현
- Function Calling 실행 주체: LLM이 판단한 JSON 데이터를 받아, 백엔드 서버에서 실제로 사내 DB를 조회하거나 타사 API를 호출하여 그 결과를 다시 AI에 전달
- 자체 모델 서빙(Self-Hosting): 비용이나 보안 문제로 오픈소스 LLM(예: Llama, Mistral)을 활용할 때, Ollama나 vLLM 등을 이용해 내부 추론 서버를 직접 구축하고 API 엔드포인트 제공
- 비용 및 성능 최적화(LLMOps): 자주 발생하는 질문을 처리하기 위한 AI 답변 캐싱(Caching) 및 무분별한 비용 발생을 막기 위한 유저별 처리율 제한(Rate Limiting) 구현
    
    ```tsx
    import { OpenAI } from 'openai';
    import { myVectorDbClient } from './vectorDb';
    
    // 1. 보안을 위해 API 키는 백엔드 환경변수에서만 관리
    const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY }); 
    
    export async function handleRAGQuery(userId: string, userQuestion: string) {
      // 2. [임베딩] 유저 질문을 숫자 벡터로 변환
      const embeddingResponse = await openai.embeddings.create({
        model: "text-embedding-3-small",
        input: userQuestion,
      });
      const questionVector = embeddingResponse.data.embedding;
    
      // 3. [Vector DB 조회] 질문 벡터와 가장 유사한 사내 문서 조각 3개 검색
      const relatedDocuments = await myVectorDbClient.search({
        vector: questionVector,
        limit: 3,
      });
      const contextText = relatedDocuments.map(doc => doc.text).join("\n");
    
      // 4. [메모리 관리] Redis에서 이 유저의 과거 대화 이력 가져오기
      const chatHistory = await getChatHistoryFromRedis(userId);
    
      // 5. [컨텍스트 주입] 사내 문서 정보와 과거 대화를 포함하여 LLM에 최종 요청
      const response = await openai.chat.completions.create({
        model: "gpt-4o",
        messages: [
          { role: "system", content: `당신은 사내 문서 기반 상담원입니다. 제공된 문서 내용만을 바탕으로 답변하세요:\n${contextText}` },
          ...chatHistory,
          { role: "user", content: userQuestion }
        ]
      });
    
      // 6. 새로운 대화를 대화 이력에 업데이트하고 프론트엔드로 결과 반환
      await saveChatHistoryToRedis(userId, userQuestion, response.choices.message.content);
      return response.choices.message.content;
    }
    
    ```