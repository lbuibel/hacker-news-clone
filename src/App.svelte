<script>
	const BASE_URL = `https://hacker-news.firebaseio.com/v0/`
	const ID_URL = `${BASE_URL}topstories.json`
	const STORY_URL = `${BASE_URL}item/`
	const TRAILING_URL = `.json?print=pretty`

	let storyIds = []
	let stories = []

	// populates the storyIds array with 30 story ID numbers
	// calling the getIds function results in a promise
	async function getIds() {
		const result = await fetch(ID_URL)
		let data = await result.json()
		if (result.ok) {
			storyIds = data
			storyIds.length = 30
			return storyIds
		} else {
			throw new Error(data)
		}
	}

	// returns 30 promises for 30 story objects
	// uses return story id from getIds
	let getStories = getIds()
		.then(res => {
			const formattedStory = res.map(async storyID => {
				const result = await fetch(`${STORY_URL + storyID + TRAILING_URL}`)
				const data = await result.json()
				if (result.ok) {
					return data
				} else {
					throw new Error(data)
			}
		})
		return formattedStory
	})

	getStories.then(story => {
		Promise.all(story)
		.then(values => {
			stories = values
		})
	})

	// convert the unix time from the api to a readable time
	function timeConverter(UNIX_timestamp){
		var a = new Date(UNIX_timestamp * 1000);
		var months = ['Jan','Feb','Mar','Apr','May','Jun','Jul','Aug','Sep','Oct','Nov','Dec'];
		var year = a.getFullYear();
		var month = months[a.getMonth()];
		var date = a.getDate();
		var hour = a.getHours();
		var min = a.getMinutes();
		var sec = a.getSeconds();
		var time = month + '. ' + date + ', ' + year + ' ' + hour + ':' + min + ':' + sec ;
		return time;
	}

	// format the URL link and clear out faulty links
	function formatLink(link) {
		console.log(link)
		if (link === undefined){
			return link = " "
		} else {
			return link.replace(/^(?:https?:\/\/)?(?:www\.)?/i, "").split('/')[0];
		}
	}

</script>

<main>
	<div class="story-container">

		<div class="story-banner">
			<h3>Hacker News Clone</h3>
		</div>

		{#each stories as story, i}
		<div class="story-post">
			<h3 class="story-index">{i + 1}.</h3>
			<div class="story-info">
				<!-- TODO - fix article link to whole article title -->
				<p class="story-author">{story.title} <span class="story-url"><a href="{story.url}" target="_blank">{formatLink(story.url)}</a></span></p>
				<p class="story-details">{story.score} points by {story.by} {timeConverter(story.time)}</p>
			</div>
		</div>

		{:else}
			<p>Loading...</p>
		{/each}
		
	</div>

</main>

<style>
	main {
		margin: 0;
		padding: 0;
		font-family: 'Quicksand', sans-serif;
	}
	p {
		margin: 0;
		padding: 0;
	}

	.story-container {
		display: flex;
		flex-direction: column;
		width: 90%;
		margin: 0 auto;
	}

	.story-banner {
		background-color: orangered;
		width: 100%;
	}

	.story-banner h3 {
		color: white;
		padding: .5rem;
		margin: 0;
		text-transform: uppercase;
	}

	.story-index {
		width: 2rem;
		align-self: center;
		padding-left: 1rem;
		font-size: 1.25rem;
		color: gray;
	}
	.story-post {
		display: flex;
		border-bottom: 1px solid gray;
	}

	.story-info {
		width: 94%;
		display: flex;
		flex-direction: column;
		justify-content: center;
	}

	.story-url a {
		color: gray;
		font-size: .85rem;
		margin-left: .5rem;
	}

	.story-url:hover {
		cursor: pointer;
	}

	.story-details {
		color: gray;
		font-size: .85rem;
	}

</style>