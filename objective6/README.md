# LLM API Lab - Objective 6

In this lab, we'll explore how to use our AI Model's API for inference, building on the earlier steps of adding API rate-limiting and authorization. By leveraging our model's API, our goal is to make the inference process easier, integrating our AI capabilities seamlessly into various applications and workflows.

## What is AI Inference

IBM Research states "inference is an AI model’s moment of truth, a test of how well it can apply information learned during training to make a prediction or solve a task... training and inference can be thought of as the difference between learning and putting what you learned into practice"[^1].

Granting access to the AI model through an API opens the door to leveraging a spectrum of progressive deployment patterns, empowering us to optimize model delivery effectively:

- Blue-Green Deployments: Facilitate seamless transitions between different versions of the model, ensuring uninterrupted service availability.
- Canary Releases: Safely introduce new model versions to a subset of users for validation before full-scale deployment.
- A/B Testing: Compare the performance of different model versions in real-world scenarios to inform decision-making.
- Feature Flags: Enable the selective activation of new model features, providing flexibility and control over functionality rollout.
- Observability: Gain insights into model performance and behavior through comprehensive monitoring and analysis.

This approach streamlines the iterative process of model development and delivery for both application developers and data scientists, fostering collaboration and enabling rapid iteration towards improved model versions. As an evolution beyond Continuous Delivery, progressive delivery allows for the evaluation of new model iterations in terms of correctness and performance, ensuring a seamless transition to enhanced AI capabilities.

## Inference API via Jupyter Notebooks

So far, we have interacted with the model API as a NetOps or developer would.  However, the data scientists on our team prefer to access the model API using the same tooling they leverage for model training and research, such as Jupyter Notebooks.  

Let's combine everything from objectives 1 - 5 together to achieve this outcome!

## Deploy Environment

We will leverage our _docker-compose.yml_ file from _objective5_ with the addition of a Jupyter Notebook container.  This file is included in the _objective6_ folder and it's contents are displayed below:

```docker
version: "3.8"
services:
  llmapi:
    image: llmapi
    ports:
      - "8080:80"
  nginx:
    image: private-registry.nginx.com/nginx-plus/base:debian
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./llmapi.jwk:/etc/nginx/llmapi.jwk
    depends_on:
      - llmapi
  jupyter:
    image: quay.io/jupyter/pytorch-notebook:latest
    ports:
      - "8888:8888"
    volumes:
      - ./notebooks:/home/jovyan/notebooks
    command: start-notebook.sh --NotebookApp.token='nginxrocks!'
    depends_on:
      - llmapi
      - nginx
```

Now we will deploy our containers:

```shell
docker compose up -d
```

## Testing

Now that our model is up and running via an API, we can leverage our Jupyter Notebook to perform inference using data the model has not seen before.

We will use UDF proxyed access as there is no jumphost in this Lab.
Please access Docker Host with method named HTTP-8888, it will open new browser tab with Jupiter UI. Put there token value _nginxrocks!_to Log In.

<img src="../objective2/HTTP-8888.png" alt="Click HTTP-8888" width="500"/>

In your Jupyter Notebook, you notice a directory called _notebooks_ in your explorer window with a notebook called _text_summarization.ipynb_.  Open the _text_summarization.ipynb_ file.

The contents of this notebook include:

```python
# the founding of F5: https://my.f5.com/manage/s/article/K245
text = """HIT Lab and Ambiente

Before co-founding F5, Mike Almquist (a.k.a Squish) was a summer intern at the HIT lab and then returned for a full-time position in 1992 after graduating from University of Delaware. He took over leadership of the software group for the lab's virtual retinal display (VRD) project. In this capacity, he created a virtual environment demo for a Virtual Worlds Consortium meeting. While Almquist and his team completed the project, they missed the deadline for the meeting, which led to his dismissal from the lab in 1994. Nevertheless, video of the demo circulated among consortium members and impressed Kubota Pacific, the company that made the graphics engines used to create the demo. The praise helped propel Almquist to his next undertaking.

After leaving the HIT lab, Almquist started a company called Ambiente to work with the UW oceanography department. The department wanted to use sonar signals from the ocean floor and turn it into a 3D graphical representation. Almquist rented a basement in an apartment on Capitol Hill for the Ambiente offices. However, the project with UW did not materialize, and, having no clients and very little money, he had to live in his basement office with only a couch to sleep on and only a sink to use as a shower.

Almquist's friend Adam Feuer thought the ocean floor mapping project was still worth pursuing without UW. In 1995, Feuer introduced Almquist to investment banker Jeff Hussey to discuss the idea for VR software for 3D maps of the ocean floor that surveyors could use. The three formed a loose partnership, and Almquist took a job as a systems administrator while working on the sea floor project in his spare time.

Make a bunch of small computers look like one big computer

After five months of research, the ocean floor project turned out to be more difficult than anticipated and the market too small to be worth pursuing further. Almquist thought VR would only be worth doing if it involved multiple users. When discussing the ideal state of how to distribute virtual environments to allow user collaboration, he said that a large, powerful server would be required, and if that's not affordable or feasible, it would be necessary to "have the ability to make a bunch of small computers look like one big computer." Hussey liked the latter idea. "Wait a minute," he said, "that could make us money!"1

Almquist decided that this work was a way to improve the world's information-delivery infrastructure so that it could handle VR content. The popularity of the internet in the late 90s led him to believe that people wanted to use it to interact with others and with media. He thought the demand was only growing and that the internet was about to become overwhelmed from the increased usage. He knew the bottleneck was not due to the transmission but to website servers. Servers can only handle so much traffic, and, if traffic gets too high, users' requests have to line up and wait for the server to process them. Almquist proposed a device that would be an intelligent, high-speed load-balancing switch. This would allow website owners to load balance connections to multiple, cheaper servers to efficiently handle increased demand rather than use one large expensive server for all traffic.

Launch of F5 Labs

The three entrepreneurs created a company, with Hussey as CEO, Feuer as VP of Engineering, and Almquist as CTO. They called their product the BIG/ip and their company Virtual Softworks, but later changed the company name to F5 Labs (F5 being the highest strength on the Fujita scale). They raised $1 million in two weeks from investors, leased office space in downtown Seattle, and started making BIG/ip.

Almquist and Feuer wanted to build up F5 Labs to the point where it could carry on without them so that they could turn it over to successors and pursue their real dream of starting a full-blown VR company. The plan was to deliver BIG/ip by the end of 1996, and Almquist estimated it would take six months to build. He envisioned a full BIG product line, consisting of BIG/ip, BIG/Firewall, a router, a server, and more BIG boxes, all installed on a BIG 19-inch rack. He called it a "VR sandbox" that would make the internet ready for delivering VR applications.

By mid-1996, the five programmers at F5 Labs had spent many months of 14-hour days "coding like the wind"2 to create the first version of BIG/ip and had installed a beta version for the busy websites of Tower Records and the ISP, Focalink. Deploying with these first customers helped the programmers work out many of the bugs in the fledgling software.

Version 1.0 of BIG/ip was set to be released in late 1996, but F5 Labs engineering missed the deadline because of a memory leak bug that caused resource issues on the BIG/ip. F5 engineers had to remotely clear BIG/ip memory on Tower's installation to keep it running until they could fix the issue.

F5 labs brought their device to the PC Magazine offices for an article featuring BIG/ip, but F5 engineers hadn't solved the memory leak bug at that point. The bug lead the PC Magazine test program to crash. The magazine thought it was their test equipment that had caused the problem so F5 engineers grabbed their gear and took off before the testers could figure it out. When the article was published, it spoke favorably of the BIG/ip software and explained it was still in beta so they didn't test it. The F5 Labs engineering team eventually created a fix in a later version.

Reorganization and strengthening for the future

Due to the delay with release and some developers leaving, F5 Labs needed to raise more money. By 1997, the company had shrunk to one developer, Almquist, and they were down to their last $200,000, which they expected would last 60 days.

From 1996 to mid-1997, Almquist built BIG/ip from version 0.9 to 1.5, along the way including features he and Hussey wanted to make it more marketable. By the time version 1.5 was released, BIG/ip was running mission-critical applications at Tower Records, one of the busiest e-commerce sites on the internet at the time. Additionally, F5 Labs received positive reviews from several publications, and Microsoft told Almquist that BIG/ip was far better than competing products they had tested.

The F5 Board of Directors hired a consultant to reorganize the company and try to get more product to the market. The consultant hired more salespeople around the country, hired more developers, hired a new VP of engineering, and moved the company to a new headquarters. The F5 Labs team, along with Almquist's friend and HIT lab alumnus, Joey King, raised enough money from investors to keep going until sales started to take off.

In January 1998, Microsoft had installed 10 BIG/ips and closed an additional $1 million order for more. Also, sales of BIG/ip crossed the $6 million mark. F5 Labs eventually dropped Labs from the company name to become F5 and rebranded the BIG/ip product as BIG-IP.

Epilogue

By mid-1998, Almquist departed F5 so that he and King could start a company with the aim of creating a platform for largescale virtual environments. Hussey remained CEO until 2000, when John McAdam took over to guide F5 through the post-dot-com bust period and beyond. The current F5 CEO, François Locoh-Donou, took the helm in 2017.3

Mike Almquist is now a winemaker and runs Almquist Family Vinters, and Jeff Hussey is currently the CEO of Tempered."""

import json, requests

token = "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiIsImtpZCI6IjAwMDEifQ.eyJuYW1lIjoiUXVvdGF0aW9uIFN5c3RlbSIsInN1YiI6InF1b3RlcyIsImlzcyI6Ik15IEFQSSBHYXRld2F5In0.ggVOHYnVFB8GVPE-VOIo3jD71gTkLffAY0hQOGXPL2I"
headers = {"Authorization": f"Bearer {token}"}
payload = {"text": text}

response = requests.post("http://nginx/summarize/", headers=headers, data=json.dumps(payload))
response.json()
```

Lets examine what is happening in this file:

- a sample text of the [founding of F5](https://my.f5.com/manage/s/article/K245).
- static JWT to authorization the API call.
- post to the _/summarize/_ API endpoint.

To execute the python code, click _shift + enter_ in each cell or click the &#x23e9; icon to restart the kernel and run all cells.

## Outcome

The results of this request are interesting.  The [Falcon AI text summarization model](https://huggingface.co/Falconsai/text_summarization) to this point has worked very well, but now we see the summary contains no information about F5.  

```json
{'summary': "Mike Almquist was a summer intern at the HIT lab . In 1992, he created a virtual environment demo for the lab's virtual retinal display (VRD) project . The project with UW did not materialize, and he had to live in his basement office with only a couch ."}
```

Our testing results help us understand this model may need additional tunning. This process involves refining our model, a task that data scientists can easily handle directly within the Jupyter Notebook. This results in an improved model ready for redeployment via our API. We repeat this tuning and redeployment cycle until the model's performance meets our standards for accuracy and reliability.

This cycle mirrors how modern application developers deploy initial microservices, then use telemetry to improve and redeploy them. By adopting progressive deployment patterns, we not only benefit our application developers but also extend these advantages to our AI-Ops teams and data scientists. This fosters collaboration and drives continuous improvement in our AI-driven projects.

## Teardown

Clean up your environment by running the following command:

```shell
docker compose down
```

## Conclusion

Throughout this lab, we've traversed a comprehensive journey, navigating through an example problem statement from inception to execution. Beginning with the discovery and testing of an AI model, we proceeded to deploy and secure our model through an API, culminating in the pivotal step of conducting inference with real-time data.

Where to go from here?

- By adding more models and NGINX locations, we can expand our capabilities and test various AI solutions. Additionally, creating an OpenAPI Spec will enhance the accessibility and compatibility of our APIs across platforms.

- Enhancing our API security with a Web Application Firewall (WAF) will proactively protect our AI models from potential threats, ensuring system integrity and security.

These examples are great problem statements to tackle leveraging your NGINX experience.  If you are new to NGINX, then check out some of our other [NGINX lab](https://clouddocs.f5.com/training/community/nginx/html/) to learn more!

[^1]: https://research.ibm.com/blog/AI-inference-explained
