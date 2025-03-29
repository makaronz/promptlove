> ⚠️ **Warning**: This project is currently in conceptual phase and not yet implemented. I welcome contributions, ideas and collaboration to help bring this vision to life! If you're interested in getting involved, please feel free to open issues, submit PRs or reach out to discuss.

# PromptMod: An AI Prompt and Agent Registry

## **Purpose**
An open-source registry for composable, reusable AI prompt templates and agents, similar to `pip`, `npm`, or `docker`. It allows developers to discover, share, and integrate modular AI components into their workflows, with built-in support for Bedrock Agents, LangChain, LangGraph, and Crew.

---

## **Goals**
- Provide reusable, modular AI prompt templates and agent configurations.
- Facilitate community-driven contributions and collaboration.
- Support interoperability with various AI frameworks and tools, including:
  - **Amazon Bedrock Agents**
  - **LangChain**
  - **LangGraph**
  - **Crew**
- First-class support for both TypeScript and Python implementations:
  - Type-safe interfaces and SDK in TypeScript
  - Pythonic API design following PEP standards
  - Cross-language compatibility guarantees
- Ensure ease of use with CLI tools, APIs, and version control.

---

## **Core Features**

### **Registry Features**
- Modular **Prompt Templates** and **Agent Configurations**.
- Metadata, tags, and versioning for easy search and compatibility.
- JSON/YAML file standards for defining templates and agents.
- Support for complex workflows and dependency graphs via **LangGraph**.

### **Installation and Distribution**
- CLI commands for installation, search, and publishing:
  - `promptmod install clarity-judge`
  - `promptmod publish`
  - `promptmod search email-classifier`

### **Interoperability**
- **Model-Agnostic**: Works with OpenAI, Claude, Amazon Bedrock, Hugging Face, and others.
- **Framework Support**:
  - **Bedrock Agents**: Native support for Amazon Bedrock's agent-based workflows.
  - **LangChain**: Simplified integration for chaining prompts and agents.
  - **LangGraph**: To

---

## Quick Start

### The Old Way

```python
import os
from langchain.document_loaders import UnstructuredEmailLoader
from langchain.chains import LLMChain
from langchain.prompts import PromptTemplate
from langchain.llms import OpenAI
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.schema import Document

# Set your OpenAI API key
os.environ["OPENAI_API_KEY"] = "your-api-key-here"

class EmailParser:
    def __init__(self, model_name="gpt-3.5-turbo", temperature=0):
        self.llm = OpenAI(model_name=model_name, temperature=temperature)
        self.text_splitter = RecursiveCharacterTextSplitter(
            chunk_size=2000,
            chunk_overlap=200
        )
    
    def load_email(self, email_path):
        """Load email from file path"""
        loader = UnstructuredEmailLoader(email_path)
        documents = loader.load()
        return documents
    
    def extract_metadata(self, email_doc):
        """Extract metadata from email document"""
        metadata = email_doc.metadata
        return {
            "sender": metadata.get("sender", ""),
            "recipient": metadata.get("recipient", ""),
            "subject": metadata.get("subject", ""),
            "date": metadata.get("date", "")
        }
    
    def parse_email_content(self, email_content, extraction_template):
        """Parse email content using LLM chain with custom template"""
        prompt = PromptTemplate(
            template=extraction_template,
            input_variables=["email_content"]
        )
        
        chain = LLMChain(llm=self.llm, prompt=prompt)
        result = chain.run(email_content=email_content)
        return result
    
    def process_email(self, email_path, extraction_template):
        """Process an email with the given extraction template"""
        # Load the email
        documents = self.load_email(email_path)
        
        if not documents:
            return {"error": "No email content found"}
        
        # Get the first document (email)
        email_doc = documents[0]
        
        # Extract metadata
        metadata = self.extract_metadata(email_doc)
        
        # Split content if it's too long
        if len(email_doc.page_content) > 2000:
            chunks = self.text_splitter.split_text(email_doc.page_content)
            # Process main chunk for now
            content = chunks[0]
        else:
            content = email_doc.page_content
        
        # Parse the content
        parsed_content = self.parse_email_content(content, extraction_template)
        
        return {
            "metadata": metadata,
            "parsed_content": parsed_content
        }


# Example usage
if __name__ == "__main__":
    # Create the parser
    parser = EmailParser()
    
    # Define extraction template based on your needs
    extraction_template = """
    From the email content below, please extract the following information:
    - Key action items
    - Meeting dates and times
    - Project deadlines
    - Important contacts mentioned
    
    Email content:
    {email_content}
    
    Extracted information:
    """
    
    # Process an email
    result = parser.process_email("path/to/your/email.eml", extraction_template)
    
    # Print the results
    print("Email Metadata:")
    for key, value in result["metadata"].items():
        print(f"{key}: {value}")
    
    print("\nParsed Content:")
    print(result["parsed_content"])

```

### The PromptMod Way

# One line to get the best community-vetted prompt for email parsing
```python
import os
import pydantic
import promptlove

pl = PromptLove(
  bias = 'performance' # (other options cost, stability)
  default_model = 'autopick'
  banned_models = 'Falcon'
)

email_extract = pl.new_task( goal = 'parse_email', input = raw_email)


```

Automatically selects best prompt from community


# That's it! Now you can focus on your actual task
results = email_parser.parse(my_email)
```

Benefits:
- ✅ Save hours of prompt engineering time
- ✅ Use battle-tested prompts from the community
- ✅ Automatic version management and updates
- ✅ Framework-agnostic - works with your existing tools

---

## Initial Prompt Examples

```
class PromptModEntry
{
  name: string;
  description: string;
  version: string;
  inputs: {};
  opts: Record<string, any>;
  return_type: ClarityJudgeResponse;
}

class PromptModResponse
{
  "args": {},
  "out" {},
  "opts": {},
  "trace": {}
}
```

## Clarity Judge

```
// *prompt-mod-def.tsx*
export const cj: PromptModEntry = {}

cj.name = "clarity-judge"
cj.description = "Judge the clarity of a statement"
cj.version = "0.0.1"
cj.prompt = "You are a clarity judge. You are given a statement and you need to judge the clarity of the statement."
cj.inputs = {prop_name: 'statement, prop_type: 'string'}
cj.opts = {
    "generate_suggestions": true,
    "audience_description": "The audience for the statement"
}
cj.return_type = ClarityJudgeResponse
```

### Clarity Judge Artifacts
```
// artifacts.ts
class Statement
{
    guid: string;
    statement: string;
    suggested_rewrite: string | null;
}
class ClarityJudgeResponse
{
  "input_variables": ["statement"],
  "output_variables": ["clarity_score"] // 0-100

  "ambiguious_statements": Statement[]
}
```

### Usage
```

import { cj } from "./promptmod/clarity-judge"
import { hl } from "@human-layer"

const example = "The technicians told the managers they made serious mistakes in their reports."

const response = await cj.invoke(example, {
    "generate_suggestions": true,
    "audience_description": "The COO of the company"
})

if (response.clarity_score < 50) {
    console.log("The statement is not clear. Here are some suggestions:")
    response.ambiguious_statements.forEach(statement => {
        console.log(statement.statement)
        console.log(statement.suggested_rewrite)
        hl.request_approval(approver, changeset);
        ...
    })
}
else {
    console.log("The statement is clear.")
}
```


## SQL Generator

```
// *prompt-mod-def.tsx*
export const sg: PromptModEntry = {}

sg.name = "sql-generator"
sg.description = "Generate a SQL statement"
sg.version = "0.0.1"
sg.prompt = "You are a SQL generator. You are given a statement and you need to generate a SQL statement."
sg.inputs = {prop_name: 'statement, prop_type: 'string'}
sg.opts = {
    "engine": "string"
    "dialect": "string"
    "schema": "string"
}

sg.return_type = SQLStatement
```

### SQL Generator
```
// artifacts.ts
class SQLStatement
{
    guid: string;
    statement: string;
}
```

### Usage
```

import { sg } from "./promptmods/sql-generator"

const example = "Get the total sales for each product"

const response = await sg.invoke(example, {
    "engine": "postgres",
    "dialect": "postgresql",
    "schema": "public"
})

console.log(response.statement)

```
## Additional Prompts Ideas to Add:
* Defensiveness Judge
* Grounding Eval
* Delegation Eval
* Writing Tone Modifier
* Actionability Eval - is this task clear enough to be actioned?
* Twitter Post Generator
* Twitter Thread Generator
* LinkedIn Post Generator
* LinkedIn Comment Generator
* Video Script Generator
* Marketing Copy Generator
* Email Generator
* Email Urgency Evaluator
* Email Tone Evaluator
* Email Clarity Evaluator
* Email Actionability Evaluator

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
