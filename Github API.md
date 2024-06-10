To create tools for interacting with GitHub functions such as comments, issues, merges, and forks, we can use the GitHub REST API. We'll define custom tools in the `src/tools` directory and integrate them into the CrewAI framework.

Here's how to create these tools:

1. **Set Up the Environment:**
   - Make sure you have the `requests` library installed.
   ```sh
   pip install requests
   ```

2. **Define Custom Tools:**
   - Create custom tools for the specific GitHub interactions you need.

### Example Tool for Managing GitHub Issues

Create a new file `src/tools/github_tools.py` and define the tools:

```python
import requests
from crewai_tools import BaseTool

class GitHubTool(BaseTool):
    base_url: str = "https://api.github.com"
    headers: dict = {
        "Accept": "application/vnd.github.v3+json",
        "Authorization": f"Bearer YOUR_GITHUB_TOKEN"
    }

    def list_issues(self, owner: str, repo: str) -> str:
        url = f"{self.base_url}/repos/{owner}/{repo}/issues"
        response = requests.get(url, headers=self.headers)
        if response.status_code == 200:
            return response.json()
        return response.text

    def create_issue(self, owner: str, repo: str, title: str, body: str) -> str:
        url = f"{self.base_url}/repos/{owner}/{repo}/issues"
        payload = {"title": title, "body": body}
        response = requests.post(url, json=payload, headers=self.headers)
        if response.status_code == 201:
            return response.json()
        return response.text

    def get_issue(self, owner: str, repo: str, issue_number: int) -> str:
        url = f"{self.base_url}/repos/{owner}/{repo}/issues/{issue_number}"
        response = requests.get(url, headers=self.headers)
        if response.status_code == 200:
            return response.json()
        return response.text

    def update_issue(self, owner: str, repo: str, issue_number: int, title: str = None, body: str = None) -> str:
        url = f"{self.base_url}/repos/{owner}/{repo}/issues/{issue_number}"
        payload = {}
        if title:
            payload["title"] = title
        if body:
            payload["body"] = body
        response = requests.patch(url, json=payload, headers=self.headers)
        if response.status_code == 200:
            return response.json()
        return response.text

    def list_comments(self, owner: str, repo: str, issue_number: int) -> str:
        url = f"{self.base_url}/repos/{owner}/{repo}/issues/{issue_number}/comments"
        response = requests.get(url, headers=self.headers)
        if response.status_code == 200:
            return response.json()
        return response.text

    def create_comment(self, owner: str, repo: str, issue_number: int, body: str) -> str:
        url = f"{self.base_url}/repos/{owner}/{repo}/issues/{issue_number}/comments"
        payload = {"body": body}
        response = requests.post(url, json=payload, headers=self.headers)
        if response.status_code == 201:
            return response.json()
        return response.text
```

### Integrate Tools into CrewAI

Update your `agents.yaml` to include the new tools:

```yaml
agents:
  - name: Chief AI Project Manager
    role: Coordinate feature development and ensure alignment with project goals
    verbose: true
    memory: true
    backstory: Leading the project with strategic vision and ensuring milestones are met.
    tools:
      - github_tools.GitHubTool
    allow_delegation: true
```

### Update Main Script

In your `src/main.py`, import and use the custom tools:

```python
import os
from crewai import Crew, Process
from configparser import ConfigParser
from github_tools import GitHubTool

# Load API keys from environment or config file
config = ConfigParser()
config.read('config.ini')

os.environ["GITHUB_TOKEN"] = config.get('github', 'token')

# Define agents, tasks, and tools here...

# Example task using GitHubTool
project_manager_task = Task(
    description='Create an issue on GitHub to track feature development progress.',
    expected_output='A GitHub issue URL.',
    tools=[GitHubTool()],
    agent=project_manager,
)

crew = Crew(
    agents=[project_manager],  # Add other agents as needed
    tasks=[project_manager_task],  # Add other tasks as needed
    process=Process.sequential
)

result = crew.kickoff(inputs={})
print(result)
```

### Configuration

Ensure you have a `config.ini` file with your GitHub token:

```ini
[github]
token = your_github_token_here
```

### Running the Project

To run the project, execute the main script:

```sh
python src/main.py
```

This setup provides you with a robust way to interact with GitHub functions through custom tools in CrewAI. You can expand these tools to cover more GitHub functionalities as needed. Let me know if you need further customization or additional features!