---
layout:     post
title:      "Exploring Open Datasets"
date:       2025-05-12 09:00:00 +0200
categories: ai
background: '/assets/figures/2025-05-12-open-datasets.jpg'
---

Last Friday was Red Hat Learning Day, and it gave me the perfect excuse to dive deeper into a topic that's been buzzing around me: Open Source AI. My investigation was particularly inspired by a webinar by Mozilla on Open Datasets, which really set the stage for my exploration. You can find more information about this webinar here: [How to Build Your Own Open Dataset](https://www.linkedin.com/events/howtobuildyourownopendataset7321254736538591234).

Now, let's be clear from the outset – the term "Open Source AI" is a bit muddled these days. So, I can't promise that the AIs I'm going to discuss are "open source" in the purest sense that some of us "old timers" might understand it. However, they offer a level of transparency and local processing that's a welcome change from many proprietary solutions.

My investigation focused on a couple of intriguing projects, particularly those that leverage open datasets based on open data principles. You can also explore other projects by the Mozilla.ai organization on their [GitHub page](https://github.com/mozilla-ai).

# Speech-to-text

My dive into speech-to-text led me to explore **Whisper**.

* **Ease of Use:** a big win for Whisper is its straightforward installation via a single Docker container. This makes it incredibly easy to get up and running.
* **Local Processing:** crucially, all data is processed locally and isn't sent to OpenAI's servers. This is a significant advantage for privacy and data sovereignty.
* **Scalability:** for those looking to integrate it into larger systems, Whisper is designed to be easily upscaled with Kubernetes.

I tested the `Systran/faster-whisper-medium` model. While it generally performs well, I encountered some peculiar issues when trying to transcribe songs. I ended up with text written in Cyrillic! This might be a specific model limitation or an edge case in its training data.

If you want to try it out, there's an online demo available: [https://huggingface.co/spaces/mozilla-ai/speaches-demo](https://huggingface.co/spaces/mozilla-ai/speaches-demo)

# Document conversion

Another area I explored was document conversion, specifically extracting content from various file formats like PDFs, DOCs, and images.

* **Installation:** this tool is easily installed via Python's `pip` package manager.
* **Local Processing:** similar to Whisper, all data processing happens locally on your machine, which is a big plus for sensitive documents. Just follow the steps on the [Github project](https://github.com/mozilla-ai/document-to-markdown)

I tested it with a presentation I had generated using a text-only LaTeX document. It seemed to work pretty nicely for the most part. However, I did notice that the OCR using the `easyocr` engine is very slow when using a CPU processing unit. To give you an idea, processing 40 slides took 250 seconds with `easyocr`, compared to a much faster 80 seconds with the default OCR recognition. Furthermore, the `easyocr` engine seemed to hallucinate many words at the end of sentences, which could be problematic for accuracy. So, yeah, I preferred the default enginer.

# More on Open Datasets

The bedrock of many of these "Open Source AI" endeavors lies in the availability of high-quality, open datasets. Understanding how to build and utilize these is crucial for the advancement of more transparent and accessible AI.

This exploration on Red Hat Learning Day was a fascinating peek into the current landscape of Open Source AI. While there are still challenges and nuances around what "open source" truly means in this context, the drive towards local processing, community involvement, and the use of open data is undoubtedly a positive step forward.
