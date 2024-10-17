# Guidelines for Writing Knowledge QNA
The **qna.yaml** file is a critical component in training a student model on custom knowledge. The instructlab knowledge tunning steps involves: creating the qna.yaml file, generating synthetic data by combining the qna file with the user-provided document, and finally, training the student model using a multi-phase InstructLab training pipeline. This guide outlines the best practices for creating a well-structured qna.yaml file for getting the best results with instructlab knowledge tunning.

The General structure of qna file for knowlege is as follow:
```
domain: 
document_outline: A one to two line description of the document
seed_examples:
  - context: <context 1 goes here>
    question_and_answers:
      - question: <question 1 goes here>
        answer: <answer 1 goes here>
      - question: <question 2 goes here>
        answer: <answer 2 goes here>
      - question: <question 3 goes here>
        answer: <answer 3 goes here>
```
The question and answer under "seed_examples" serve as a reference for the teacher model to generate additional similar question-answer pairs based on the provided document. Therefore, the quality of the question-answer pairs, their groundedness in the context, and how well the document outline is crafted will directly impact the quality of the synthetic data generated. This, in turn, will effect the student model performance on user document.

### **Question-Answer Pair:**
1. Make the question-answer pairs more detailed
	1. Avoid single-word answers. Even if the question has a one-word answer, provide a full sentence, as this sets the style for how the teacher model will generate similar QA pairs.
	2. Makesure the answers are directly relevant to the questions.
    ```
    Example of irrelevant QA
    ```
	4. Each question should be self-contained, including all necessary information, and not rely on any additional context (such as multi-turn chat history).
	e.g. This is an example of bad question answer
    ```
    - question: What is function of attention mask?
      answer: Function of the attention mask in a transformer model...
    - question: How is it implemented?
      answer: It is implemented as QKV matrix...
    
    ```
2. Match the QA to the downstream evaluation:
	1. It is important to keep the style of question and answer similar to the dowstream task or how the model is going to be evaluated. 
	```
    Give few examples of downstream task and their QA
    ```
	3. For example a model trained on product manual that will be used in customer care setting will have question relating customer seeking help for specific failure scenarios.
	4. If a evaluation set if available design Question and Answer to be similar to the evaluation questions.
3. QA should be grounded in the given context:
	1. Make-sure the context and QA are from same document and Question and Answer directly address information mentioned in the context
	2. Maksure the context includes assumed terms in QA:
	    - For example question and answer might reference Financial statement of RBC bank but the context contains a exert from Financial statement  without actually mentioning RBC. This is a common scenario where chunks might not always reference the title or might be missing enough context information to be understood on their own.
4. Check the accuracy of answers for given question:
	- Check if the any cited numerical information are correct given the context. For example: performance numbers, percentage, ratios, amount etc mentioned in the context is accurately cited. Avoid any conversion of units, e.g. if context mentions 10 billion then avoid using 10,000 million in answer. 
	- If the answer involves a number that is inferred through some reasoning process mention those reasoning steps before giving the final answer. Also verify the reasoning steps and final answer.
	- If the information in the answer is of abstractive in nature, ensure it can be properly inferred from given context. In cases where it is not very clear it is better to simply avoid those type of questions
	- Ensure the answer is complete. All the information that is part of answer needs to be in the answer. For example if the answer is set of steps ensure all the steps from context are mentioned.
### **Document Outline:**
The document outline is crucial in synthetic data generation, as it provides additional context for each section of the knowledge document, ensuring the generated data is more relevant and better aligned for knowledge tuning.

Here are different cases you might encounter when creating document outline:
1. If the document is a  textbook/article: 
    - You can use the textbook title as document outline.  Include any information from the document that makes the title more specific. For example: `Physics textbook vol 1` vs `Physics textbook vol 1: Vector Mathematics, Laws of Motion, and Gravitation`. This added specificity ensures that the title gives a clearer context for each section.
2. If the document is manual:
	- You can use the title as document outline. Similar to 1. make the title specific if possible. Expand any abbreviations mentioned in the title. For example, instead of `SOP of Company A` expand the name to be `Standard Operating Procedure (SOP) of Company A`
3. If the document is a yearly published document:
	- If the document is published yearly or has specific edition associated with it include that information in the document outline. For example: IBM Annual Report vs IBM Annual Report 2024
4. If the document is data sheet, tables or any other form of data: Include gist of what the data represents. For example: If the document includes nations, their GDP, population density, size etc. then title could be `World GDP, Census and Geographical Data`
### **Extracting Context**:
Under seed_examples you will be required to provide atleast 3 context/text spans taken from the user document or very similar document.


1. These context along with the question and answers based on this respective context will provide teacher model guidance on generating synthetic data for training
2. The context should:
    - Preferably comes from the user document or any very similar document.
    - Should include all necessary information mentioned in the question and answer that follow it. For example avoid: If the passage talks about generic card processing but the QA mention specific bank names.
    - Should be within approximately 300-500 words.
    - If the context includes table then ensure the table markdown is valid, includes all the necessary rows/columns, table headers, any necessary caption etc.
3. When selecting context from a user document, we recommend choosing a diverse range of text to ensure comprehensive coverage of the document's various elements.
    - For instance, if the document contains definitions, theorems, tables, process steps, narrative sections, etc., aim to include contexts that represent as many of these different types as possible.
