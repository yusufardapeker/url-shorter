<script setup>
import { ref } from "vue";
import Button from "./shared/Button.vue";
import bgShortenMobileImg from "@/images/bg-shorten-mobile.svg";
import bgShortenDesktopImg from "@/images/bg-shorten-desktop.svg";
import clsx from "clsx";

const hasInputError = ref(false);
const inputErrorMessage = ref("");
const fullUrl = ref("");
const shortenedUrls = ref([]);
const copiedUrl = ref("");
const loading = ref(false);

const checkFullUrl = () => {
	if (fullUrl.value.trim() === "") {
		hasInputError.value = true;
		inputErrorMessage.value = "Please add a link";

		return false;
	} else if (!fullUrl.value.trim().includes("https://")) {
		hasInputError.value = true;
		inputErrorMessage.value = "Added link is invalid";

		return false;
	} else if (shortenedUrls.value.some((url) => url.original === fullUrl.value.trim())) {
		hasInputError.value = true;
		inputErrorMessage.value = "Please add a different link for shortening";

		return false;
	} else if (shortenedUrls.value.some((url) => url.short === fullUrl.value.trim())) {
		hasInputError.value = true;
		inputErrorMessage.value = "This link is already shortened!";

		return false;
	} else {
		hasInputError.value = false;
		return true;
	}
};

const shortenUrl = async () => {
	if (checkFullUrl()) {
		try {
			loading.value = true;

			const res = await fetch("https://api.tinyurl.com/create", {
				method: "POST",
				headers: {
					Authorization: `Bearer ${import.meta.env.VITE_API_KEY}`,
					"Content-Type": "application/json",
				},
				body: JSON.stringify({ url: fullUrl.value }),
			});

			const data = await res.json();

			console.log("API response:", data);

			shortenedUrls.value = [
				{
					original: fullUrl.value,
					short: data.data.tiny_url,
				},

				...shortenedUrls.value,
			];

			loading.value = false;
			fullUrl.value = "";
		} catch (error) {
			console.log("Fetch error: ", error);
		}
	}
};

const handleCopyUrlButtonsStyle = (e) => {
	const targetButton = e.target;
	const allButtons = document.querySelectorAll(".btn");

	allButtons.forEach((button) => button.classList.remove("secondary"));
	targetButton.classList.add("secondary");
};

const copyUrl = (e, url) => {
	handleCopyUrlButtonsStyle(e);

	navigator.clipboard.writeText(url);
	copiedUrl.value = url;
};
</script>

<template>
	<section id="url-shorter">
		<div class="url-shorter-wrapper container">
			<div class="actions">
				<picture>
					<source media="(min-width: 376px)" :srcset="bgShortenDesktopImg" />
					<img :src="bgShortenMobileImg" alt="bg-img" class="bg-shorten-img" />
				</picture>

				<input
					type="text"
					:class="clsx('url-input', { error: hasInputError })"
					placeholder="Shorten a link here..."
					v-model="fullUrl"
				/>
				<span v-if="hasInputError" class="error-message">{{ inputErrorMessage }}</span>
				<Button variety="cornered" :onClickFunction="shortenUrl">Shorten It!</Button>
			</div>

			<div v-if="loading" class="loading">
				<span class="circle"></span>
				<span class="triangle"></span>
				<span class="square"></span>
			</div>

			<ul class="shortened-urls">
				<li v-for="url in shortenedUrls" class="links-wrapper" :key="url.short">
					<span class="full-url">{{ url.original }}</span>

					<div class="shortened-link">
						<span class="link">{{ url.short }}</span>
						<Button variety="cornered" :onClickFunction="(e) => copyUrl(e, url.short)">
							{{ copiedUrl === url.short ? "Copied!" : "Copy" }}</Button
						>
					</div>
				</li>
			</ul>
		</div>
	</section>
</template>
