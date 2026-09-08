# AIUI Studio Global QuickStart 
## I. What Is AIUI Studio?
AIUI Studio(Global) is Rokid's **one-stop AIUI agent development and build platform** for Rokid Glasses. It runs in a browser, so no local development environment is required. It provides developers with the complete workflow from **creating an AIUI agent** to **submitting it for publication**:

+ Develop with **AIUI CODING**: describe requirements in natural language, and AI directly reads and writes project code;
+ **Real-device simulation**: simulate how an agent interacts on the glasses, or run the agent directly;
+ **Agent debugging and submission**: complete real-device verification, generate a version, save the required information, and submit it for review.

## II. Sign In to AIUI Studio
1. Open AIUI Studio(Global) in a browser: [https://aiui-global.rokid.com/](https://aiui-global.rokid.com/)
2. If you are not signed in, complete sign-in through the Rokid Account Center.
3. After signing in, you will be returned to the workspace automatically. The agent list on the left will load the cloud-based AIUI agents associated with your account.

## III. Create an AIUI Project in AIUI Studio(Global)
There are three ways to create a new AIUI agent:

| Creation method | Applicable scenario | Result |
| :---: | :---: | :---: |
| Create with Coding | Build an agent from scratch | Enter AIUI CODING and generate a complete project with natural language |
| Import form local folder | An AIUI project already exists on your computer | Authorize the system folder and import the code |
| Import from Github | The code is in a remote repository | Import the specified directory by repository URL, branch, or tag |

⚠️ Choose only one method to create an AIUI project. Beginners are recommended to use Conversational creation.


## Method 1: Create with Coding
AIUI CODING is the primary development interface. The first time you use it, download and enable the built-in `aiui-dev` Skill in AIUI Studio.

![image.png](../../image/quickstart.en-us/1.png)

You can continue to use natural language to add features, adjust the interface, or troubleshoot issues. AI reads the project context and directly modifies project files.

## Method 2: Import form local folder (Create an AIUI Scaffold with npm)
**(1) Enter the following in a device terminal (or command prompt):**

```plain
npm create @yodaos-pkg/aiui-agent@latest my-agent
```

```plain
cd my-agent  # Enter the folder named my-agent
ls           # List files and subfolders in the current folder
```

![image.png](../../image/quickstart.en-us/2.png)

**(2) Locate the directory containing the files.**

![image.png](../../image/quickstart.en-us/3.png)

**(3) Click “Local import” and select the corresponding local folder.**

![image.png](../../image/quickstart.en-us/4.png)

## Method 3: Import from Github
AIUI Sample project: [https://github.com/jsar-project/AIUI/tree/main/samples](https://github.com/jsar-project/AIUI/tree/main/samples)

![image.png](../../image/quickstart.en-us/5.png)

## IV. Generate and Modify Projects with AIUI CODING
AIUI CODING is AIUI Studio's AI development tool. You can continuously use natural language to add project features, adjust the interface, or troubleshoot issues. The AI CODING tool reads the project context and directly modifies project files.

+ **Command input box**: Enter a requirement and send it; generation can be stopped at any time.

![image.png](../../image/quickstart.en-us/6.png)

+ **Context attachments**: Attach relevant files to the current instruction to help AI accurately understand the scope of the change.

![image.png](../../image/quickstart.en-us/7.png)

+ **Instruction suggestions:** Describe one clear objective per instruction, and include the page state, interaction method, and acceptance criteria.

![image.png](../../image/quickstart.en-us/8.png)

## V. AIUI Agent Web-Based Simulation and Debugging
Click the **“Preview”** button in the **“Device Simulation”** section to debug through a web-based simulation. You can simulate user input, temple controls, and how the agent is displayed under different lighting conditions.

![image.png](../../image/quickstart.en-us/9.png)

**[Voice input]** Simulates the complete process from wake-up, speech recognition, and the large language model through voice playback.

**[Temple Controls]** Simulates glasses interactions including click, double-click, swipe backward, and swipe forward.

**[Environment Simulation]** Simulates how the AIUI agent appears in different environments.

## VI. Debug an AIUI Agent on a Real Device
Package the AIUI agent through **Build & Review -> Package AIX**. After packaging succeeds, the package is automatically synchronized to the cloud.

![image.jpg](../../image/quickstart.en-us/10.png)

> ⚠️ Before debugging an AIUI agent on a real device, you must complete the **Package AIX** step to obtain the latest version of the AIUI agent.

In the Hi Rokid app, go to **Settings -> Developer -> Update glasses resource package**.

![image.png](../../image/quickstart.en-us/11.png)

After you see the message **“Agent resource package downloaded successfully,”** invoke the agent through semantic matching and experience the complete real interaction flow.

For example: “Hi Rokid, open the xxx agent.”

![image.png](../../image/quickstart.en-us/12.png)

## VII. View and Edit AIUI Agent Code
Open the **“Code”** tab on the right to inspect or manually edit AIUI CODING. The file tree supports creating, renaming, deleting, copying paths, and refreshing directories.

![image.png](../../image/quickstart.en-us/13.png)

An AIUI agent's directory structure usually includes global configuration, Pages, Widgets, Agent Workers, components, and assets:

```latex
agent-app/
├── AGENTS.md
├── app.json
├── app.js
├── pages/
│   └── index/
│       └── index.ink
├── widgets/
│   └── weather/
│       └── index.ink
├── workers/
│   └── sync.js
└── assets/
```

+ `AGENTS.md`: Describes the agent's identity, responsibilities, and behavioral boundaries.
+ `app.json`: Declares Pages, Widgets, Agent Workers, and global window behavior.
+ `pages/`: Stores pages and interaction logic.
+ `widgets/`: Stores compact independent interfaces, with one `.ink` file for each entry.
+ `workers/`: Stores Agent Worker background scripts with no interface.
+ `assets/`: Stores static resources such as images and audio.
+ `.ink`: A file format that combines Page or Widget configuration, logic, structure, and styles in a single file.
+ A project can also use a multi-file page structure composed of WXML, WXSS, JavaScript, and JSON.

For more information about AIUI file and page structures, see [Project structure](../structure.en-US.md).

## VIII. Publish and Submit for Review in the Hi Rokid Agent Store
In the **“Build &Review”** section, fill in the basic information, permission dependencies, and preview materials. Then save the information and submit it for review.

![image.png](../../image/quickstart.en-us/14.png)

After confirming that the information and current version are correct, click **“Submit for review.”**

Review statuses:

+ **Under review**: Wait for platform review. You can view the status in the agent list during this period.
+ **Review rejected**: Modify the code or information according to the rejection reason, then repackage, save, and resubmit.
+ **Review approved**: The agent enters a publishable state and can be displayed and used in the agent store in the Rokid AI app.

Before submitting, it is recommended that you confirm once more that the core real-device flow has passed, permission declarations match the code, the information contains no exaggerated claims, and the number and formats of images and videos meet the requirements.
