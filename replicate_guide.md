Script example of LLM inference using replicate.com platform.

Please strictly follow the model type, which is "openai/gpt-5" (yes, it's already there).
And follow the other parameters value provided in the example: "max_completion_tokens", "reasoning_effort"

```py
# /// script
# requires-python = ">=3.13"
# dependencies = [
#     "replicate",
# ]
# ///
import replicate

system_prompt = """
You are critical and honest. You know by being honest, it will be better for the user.
"""

prompt = "Focus on my answers in qna section. Are they good given the application context?"

with open("vacancy.md", "r") as f:
    vacancy = f.read()
with open("qna.md", "r") as f:
    qna = f.read()

prompt += "\n<vacancy>\n" + vacancy + "\n</vacancy>\n"
prompt += "\n<letter>\n" + letter + "\n</letter>\n"

output = replicate.run(
    "openai/gpt-5",
    input={
        "system_prompt": system_prompt,
        "prompt": prompt,
        "max_completion_tokens": 128000,
        "reasoning_effort": "medium",
        # "temperature": 0.3,
        # "image_input": "" #image_url,
    },
)

print("".join(output))
```