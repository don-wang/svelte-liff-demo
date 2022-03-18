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
			errorMessage = 'アプリを利用するにはLINEで開く必要があります';
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
										url: 'https://i.pinimg.com/originals/97/05/62/970562dc32d225014f2b4d1ee9dc008c.png',
										margin: 'none',
										offsetTop: 'md',
										offsetBottom: 'md'
									},
									{
										type: 'image',
										url: 'https://mitsui-shopping-park.com/lalaport/common/image/svg/logo.svg',
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
								text: 'ららぽーと福岡行き無料券',
								weight: 'bold',
								size: '28px',
								margin: 'xxl',
								align: 'center',
								wrap: false,
								color: '#275aa3'
							},
							{
								type: 'text',
								text: '西鉄大橋駅 → ららぽーと福岡駅',
								align: 'center',
								margin: 'xxl',
								size: 'xl',
								color: '#275aa3'
							},
							{
								type: 'text',
								text: `${today} 最終バスまで有効`,
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
										text: '降車時に乗務員が確認します。',
										wrap: true,
										align: 'center'
									},
									{
										type: 'text',
										text: 'しっかりご提示ください。',
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
								<Label>チケット発券</Label>
							</Button>
						</Group>
						<Group>
							<Button on:click={closeWindow} variant="raised" class="button-shaped-round">
								<Label>閉じる</Label>
							</Button>
							<Button
								on:click={sendMessage}
								disabled
								variant="raised"
								class="liffBtn button-shaped-round"
							>
								<Label>発券済み</Label>
							</Button></Group
						>
					</Content>
				</Card>
			</div>
		</Cell>
		<Cell span={12}>
			<div class="card-container">
				<Card>
					<Content>
						<h2>ユーザー情報</h2>
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
						<h2>LIFF 環境情報</h2>
						<List twoLine nonInteractive>
							<Item>
								<Text>
									<PrimaryText>言語設定</PrimaryText>
									<SecondaryText>
										{liffInfo.lang}</SecondaryText
									>
								</Text>
							</Item>
							<Item>
								<Text>
									<PrimaryText>LIFFバージョン</PrimaryText>
									<SecondaryText>{liffInfo.ver}</SecondaryText>
								</Text>
							</Item>
							<Item>
								<Text>
									<PrimaryText>LIFFブラウザ</PrimaryText>
									<SecondaryText>{liffInfo.isInClient}</SecondaryText>
								</Text>
							</Item>
							<Item>
								<Text>
									<PrimaryText>ログイン</PrimaryText>
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
									<PrimaryText>LINEバージョン</PrimaryText>
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
