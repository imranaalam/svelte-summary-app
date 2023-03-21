<script>
  import { onMount } from "svelte";
  import Markdown from "svelte-markdown";
  let inputUrls = "";
  let summaryPoints = [];
  let article = "";
  let progress = "";

  async function fetchSummaries() {
    progress = "Fetching summaries...";
    const urls = inputUrls.split("\n").filter(url => url.trim() !== "");
    for (const url of urls) {
      try {
        const response = await fetch("https://api.openai.com/v1/completions", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
            "Authorization": "Bearer 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",
          },
          body: JSON.stringify({
            model: 'text-davinci-003',
            prompt: `Summarize the following article in bullet points: ${url}`,
            max_tokens: 3000,
            temperature: 0.5 ,
          }),
        });
        const data = await response.json();
        summaryPoints = [...summaryPoints, data.choices[0].text.trim()];
        console.log(summaryPoints)
      } catch (error) {
        console.error("Error fetching summary:", error);
      }
    }
    await generateArticle();
  }

  async function generateArticle() {
    progress = "Generating article...";
    const prompt = `Write an 800-word article with headings based on these summary points:\n${summaryPoints.join("\n")}`;

    try {
      const response = await fetch("https://api.openai.com/v1/completions", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          "Authorization": "Bearer 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",
        },
        body: JSON.stringify({
          model: 'text-davinci-003',
          prompt: prompt,
          max_tokens: 3000,
          temperature: 0.5 ,
        }),
      });
      const data = await response.json();
      article = data.choices[0].text.trim();
      console.log(article)
    } catch (error) {
      console.error("Error generating article:", error);
    }
    progress = "";
  }
</script>

<style>
@import "bulma/css/bulma.min.css";
  main {
    padding: 2rem;
  }
</style>


<main class="section">
	<div class="container">
	  <div class="columns is-centered">
		<div class="column is-half">
		  <div class="box">
			<h1 class="title is-1">Article Summarizer and Generator</h1>
			<div class="field">
			  <label for="urls" class="label">Enter URLs (one per line):</label>
			  <div class="control">
				<textarea class="textarea" bind:value="{inputUrls}" placeholder="Enter URLs (one per line)"></textarea>
			  </div>
			</div>
			<div class="field">
			  <div class="control">
				<button class="button is-primary" on:click="{fetchSummaries}">Generate Summary and Article</button>
			  </div>
			</div>
		  </div>
		</div>
	  </div>
	  {#if progress}
		<section class="section">
		  <div class="container">
			<h2 class="subtitle">{progress}</h2>
			<progress class="progress is-primary" max="100"></progress>
		  </div>
		</section>
	  {/if}
	  {#if summaryPoints.length > 0}
		<section class="section">
		  <div class="container">
			<h2 class="subtitle is-3">Summary Points:</h2>
			<div class="content">
			  <Markdown source="{summaryPoints.join('\n\n- ')}" />
			</div>
		  </div>
		</section>
	  {/if}
	  {#if article}
		<section class="section">
		  <div class="container">
			<h2 class="subtitle is-3">Generated Article:</h2>
			<div class="content">
			  <Markdown source="{article}" />
			</div>
		  </div>
		</section>
	  {/if}
	</div>
  </main>