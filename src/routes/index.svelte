<script>
	import Geolocation from 'svelte-geolocation';
	import moment from 'moment';
	// optional import focus-visible polyfill only once
	import 'focus-visible';
	import Button, { Group, Label, Icon } from '@smui/button';
	import LayoutGrid, { Cell } from '@smui/layout-grid';
	import Card, { Content } from '@smui/card';
	import List, { Item, Graphic, Meta, Text, PrimaryText, SecondaryText } from '@smui/list';
	import { onMount } from 'svelte';
	let liff;
	let coords = [];
	let accessToken;
	let userInfo = {};
	let errorMessage = '';
	let liffInfo = {
		lang: '',
		ver: '',
		isInClient: '',
		isLoggedIn: '',
		OS: '',
		lineVer: ''
	};

	onMount(async () => {
		const l = await import('@line/liff');
		liff = l.default;
		liff
			.init({
				liffId: '1656984897-84oW2nrm'
				// withLoginOnExternalBrowser: true
			})
			.then(() => {
				liffInfo.lang = liff.getLanguage();
				liffInfo.ver = liff.getVersion();
				liffInfo.isInClient = liff.isInClient();
				liffInfo.isLoggedIn = liff.isLoggedIn();
				liffInfo.OS = liff.getOS();
				liffInfo.lineVer = liff.getLineVersion();

				if (liff.isLoggedIn()) {
					liff
						.getProfile()
						.then((profile) => {
							userInfo = profile;
						})
						.catch((err) => {
							console.log('error', err);
						});
					accessToken = liff.getAccessToken();
				}
			})
			.catch((err) => {
				errorMessage = err;
				console.log(err);
			});
		if (!liff.isInClient()) {
			errorMessage = 'You need to open this link in line';
		}
	});

	const sendMessage = () => {
		const today = moment().format('MM/DD');
		const message = [
			{
				type: 'flex',
				altText: 'this is a flex message',
				contents: {
					type: 'bubble',
					size: 'giga',
					body: {
						type: 'box',
						layout: 'vertical',
						contents: [
							{
								type: 'box',
								layout: 'horizontal',
								contents: [
									{
										type: 'image',
										url: 'https://d.line-scdn.net/n/_s1/_0/linecorp-web-uit/images/line_icon_200_v3.jpg',
										margin: 'none',
										offsetTop: 'md',
										offsetBottom: 'md'
									},
									{
										type: 'image',
										url: 'https://upload.wikimedia.org/wikipedia/commons/thumb/1/1b/Svelte_Logo.svg/199px-Svelte_Logo.svg.png',
										margin: 'none',
										offsetTop: 'md',
										offsetBottom: 'md',
										offsetStart: 'none'
									}
								],
								margin: 'none'
							},
							{
								type: 'text',
								text: 'Have a good day',
								align: 'center',
								margin: 'xxl',
								size: 'xl',
								color: '#275aa3'
							},
							{
								type: 'text',
								text: `Today is ${today} `,
								align: 'center',
								margin: 'xl',
								size: 'xl',
								color: '#eb336d'
							},
							{
								type: 'separator',
								margin: 'xxl'
							},
							{
								type: 'box',
								layout: 'vertical',
								contents: [
									{
										type: 'text',
										text: 'Good luck with Svelte and LIFF',
										wrap: true,
										align: 'center'
									}
								],
								borderColor: '#D3D3D3',
								borderWidth: 'normal',
								cornerRadius: 'sm',
								paddingAll: 'lg',
								margin: 'xxl'
							}
						]
					},
					styles: {
						footer: {
							separator: true
						}
					}
				}
			}
		];

		if (!liff.isInClient()) {
			window.alert(errorMessage);
		} else {
			liff
				.sendMessages(message)
				.then(() => {
					liff.closeWindow();
				})
				.catch((error) => {
					window.alert('Error sending message: ' + error);
				});
		}
	};

	const closeWindow = () => {
		if (!liff.isInClient()) {
			window.alert(errorMessage);
		} else {
			liff.closeWindow();
		}
	};
</script>

<svelte:head>
	<title>LIFF Demo</title>
</svelte:head>
<Geolocation getPosition bind:coords />
<div class="content">
	<h1>Liff Demo</h1>
	<h2 class="error">{errorMessage}</h2>
	<LayoutGrid>
		<Cell span={12}>
			<div class="card-container">
				<Card>
					<Content>
						<Group>
							<Button on:click={() => sendMessage()} variant="raised" class="button-shaped-round">
								<Label>Send Message</Label>
							</Button>
						</Group>
						<Group>
							<Button on:click={closeWindow} variant="raised" class="button-shaped-round">
								<Label>Close</Label>
							</Button>
						</Group>
					</Content>
				</Card>
			</div>
		</Cell>
		<Cell span={12}>
			<div class="card-container">
				<Card>
					<Content>
						<h2>User Info</h2>
						<List twoLine nonInteractive>
							<Item>
								<Graphic class="material-icons">gps_fixed</Graphic>
								<Text>
									<PrimaryText>GPS</PrimaryText>
									<SecondaryText>{JSON.stringify(coords)}</SecondaryText>
								</Text>
							</Item>
						</List>
					</Content>
				</Card>
				<Card>
					<Content
						><h4>userId:</h4>
						<p>{userInfo.userId}</p></Content
					>
				</Card>
				<Card>
					<Content
						><h4>accessToken:</h4>
						<p>{accessToken}</p></Content
					>
				</Card>
			</div>
		</Cell>
		<Cell span={12}>
			<div class="card-container">
				<Card>
					<Content>
						<h2>LIFF Info</h2>
						<List twoLine nonInteractive>
							<Item>
								<Text>
									<PrimaryText>Language</PrimaryText>
									<SecondaryText>
										{liffInfo.lang}</SecondaryText
									>
								</Text>
							</Item>
							<Item>
								<Text>
									<PrimaryText>LIFF Ver</PrimaryText>
									<SecondaryText>{liffInfo.ver}</SecondaryText>
								</Text>
							</Item>
							<Item>
								<Text>
									<PrimaryText>LIFF Browser</PrimaryText>
									<SecondaryText>{liffInfo.isInClient}</SecondaryText>
								</Text>
							</Item>
							<Item>
								<Text>
									<PrimaryText>Login Status</PrimaryText>
									<SecondaryText>{liffInfo.isLoggedIn}</SecondaryText>
								</Text>
							</Item>
							<Item>
								<Text>
									<PrimaryText>OS</PrimaryText>
									<SecondaryText>{liffInfo.OS}</SecondaryText>
								</Text>
							</Item>
							<Item>
								<Text>
									<PrimaryText>LINE Ver</PrimaryText>
									<SecondaryText>{liffInfo.lineVer}</SecondaryText>
								</Text>
							</Item>
						</List>
					</Content>
				</Card>
			</div>
		</Cell>
	</LayoutGrid>
</div>

<style>
	.content {
		margin: 0 10px;
	}
	.error {
		color: red;
	}
</style>
