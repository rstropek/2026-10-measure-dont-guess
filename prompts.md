Run an eval for the quizzes in chapter 4. The model we want to test is Gemma 4 hosted at SCCH. The model that should judge the response of Gemma 4 is gpt-6.1-sol (Azure Foundry).

In parallel, run the same quiz eval (gpt-6.1-sol on Azure Foundry as judge) through my local Novedu checkout, with the grader model nvidia/Gemma-4-26B-A4B-NVFP4 running on a freshly rented RTX 5090 through vast.ai, ideally in Europe. You can remove the rented machine once the eval is completed.

Give me a comparison of the outcomes (results, provision time for vast.ai, and runtimes/API times) at the end. Write it in the folder `2026-10-15-comparison` using Markdown. Add meaningful charts so that I get an understanding of the results quickly. Convert the report into PDF, too.
