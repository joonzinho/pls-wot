<script lang="ts">
	import { ReviewEvent } from "$lib";
	import { relayList, relayPool } from "$lib/nostr";
	import { npubEncode } from "nostr-tools/nip19";
	import { onMount } from "svelte";

	let ratings: {
		from: string;
		to: string;
		date: number;
		score: boolean;
		businessAlreadyDone: boolean;
		description: string;
	}[] = [];

	$: ratings.sort((a, b) => b.date - a.date);

	onMount(() => {
		relayPool.subscribeMany(
			relayList,
			[
				{
					kinds: [ReviewEvent],
					"#l": ["pls-wot-rating"],
				},
			],
			{
				onevent(e) {
					try {
						const c = JSON.parse(e.content);

						const newRating = {
							from: npubEncode(c.from),
							to: npubEncode(c.to),
							date: e.created_at * 1000,
							score: c.score,
							businessAlreadyDone: c.businessAlreadyDone,
							description: c.description,
						};
						
						if (ratings.find((r) => r.from === newRating.from && r.to === newRating.to)) {
							ratings = ratings.filter((r) =>
								!(
									r.from === newRating.from
									&& r.to === newRating.to
								)
							)
						}

						ratings = [...ratings, newRating]

					} catch (error) {
						console.error("Error processing the event:", error);
					}
				},
			}
		);
	});
</script>

<div class="flex items-center flex-col gap-8">
	<h1 class="text-2xl font-bold">Ratings table (Currently using replaceable events)</h1>

	<table>
		<thead>
		<tr>
			<th>Rater Nostr Key</th>
			<th>Rated Nostr Key</th>
			<th>Date</th>
			<th>Rating</th>
			<th>Had <br> business</th>
			<th>Description</th>
		</tr>
		</thead>
		<tbody>
		{#each ratings as rating}
			<tr>
				<td>{rating.from}</td>
				<td>{rating.to}</td>
				<td>
					{new Date(rating.date).toLocaleDateString()}
					<br />
					{new Date(rating.date).toLocaleTimeString()}
				</td>
				<td>{rating.score ? "👍" : "👎"}</td>
				<td>{rating.businessAlreadyDone ? "👍" : "👎"}</td>
				<td>{rating.description}</td>
			</tr>
		{/each}
		</tbody>
	</table>
</div>

<style>
    table, th, td {
        border: 1px solid;
    }

    td {
        text-align: center;
    }

    table {
        width: 80%;
    }
</style>