# exercise-prescription-agent

LangGraph 기반 멀티 에이전트 운동 처방 시스템. 사용자의 신체 고민을 입력하면 세 개의 에이전트가 맞춤형 운동 처방 리포트를 생성

---

# 멀티 에이전트 구성

이 프로젝트는 단일 LLM 호출로 모든 것을 처리하는 방식 대신, 역할이 분리된 세 개의 에이전트를 LangGraph로 연결하여 파이프라인을 구성 각 에이전트는 자신의 역할에만 집중, 상태(State) 객체를 통해 다음 에이전트로 결과를 전달

```
사용자 입력 → [extractor] → [matcher] → [answer] → 최종 리포트
```

### extractor — 증상 추출 에이전트

사용자가 자연어로 입력한 건강 고민에서 핵심 명사만 추출한다. 불필요한 조사와 서술어를 제거하여 다음 에이전트가 처리하기 쉬운 형태로 정제

##### 입력 예시
```
체력이 안좋고 살이 계속찌는데 어떤운동을 할까
```

##### 출력 예시
```
체력 저하, 체중 증가
```

---

### matcher — 운동 매칭 에이전트

추출된 증상을 바탕으로 전문 지식 기반의 운동 5가지를 선정하고, 각 운동의 핵심 효과를 한 문장으로 정리

##### 입력 예시
```
체력 저하, 체중 증가
```

##### 출력 예시
```
인터벌 러닝 - 심폐 기능 강화 및 지방 연소
스쿼트 - 하체 근력 강화 및 기초대사량 증가
플랭크 - 코어 안정성 및 전신 밸런스 개선
유산소 운동 - 체지방 감소 촉진
요가 - 유연성 향상 및 정신적 안정
```

---

### answer — 리포트 생성 에이전트

앞선 두 에이전트의 결과를 받아 건강 검진 리포트 형식으로 최종 출력물을 생성한다. 마크다운 기호 없이 명사형으로만 종결하는 출력 규칙을 프롬프트 수준에서 강제

---

# 그래프 구조

LangGraph의 `StateGraph`로 구성한 선형 파이프라인. 각 노드는 에이전트 함수이며, 공유 상태 객체(`ExerciseState`)를 통해 데이터를 전달

<table>
<tr>
<td valign="middle" width="30%">
<img width="108" height="432" alt="75b374dc-c61e-43a1-88f4-006a986ffb33" src="https://github.com/user-attachments/assets/f6495b0c-baec-4d97-a54f-29c604aa46ac" />
</td>
<td valign="middle" width="70%">
<pre><code class="language-python">workflow = StateGraph(ExerciseState)

workflow.add_node("extractor", extractor_agent)
workflow.add_node("matcher", matcher_agent)
workflow.add_node("answer", answer_agent)

workflow.set_entry_point("extractor")
workflow.add_edge("extractor", "matcher")
workflow.add_edge("matcher", "answer")
workflow.add_edge("answer", END)
</code></pre>
</td>
</tr>
</table>

---

# 실행 결과

<img width="852" height="422" alt="image" src="https://github.com/user-attachments/assets/ba32bc2e-31d6-4e03-9caa-73c39b05f716" />


---

# 기술 스택

| 항목 | 내용 |
|------|------|
| 언어 | Python 3.11 |
| 오케스트레이션 | LangGraph |
| LLM 프레임워크 | LangChain |
| 로컬 LLM | Ollama — LG EXAONE 3.5 2.4B |

---

# 실행 방법

```bash
# Ollama에 모델 설치
ollama pull exaone3.5:2.4b

# 의존성 설치
pip install langchain langchain-community langgraph

# 실행
python main.py
```

`input_query` 변수에 원하는 건강 고민을 입력

```python
input_query = "허리가 자주 아프고 어깨가 뭉치는데 어떤 운동이 좋을까"
```
