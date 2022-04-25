<script>
  import liff from "@line/liff";
  import Geolocation from "svelte-geolocation";
  import moment from "moment";
  import LIFFInspectorPlugin from "@line/liff-inspector";
  import { LiffMockPlugin } from "@line/liff-mock";

  import { onMount } from "svelte";

  liff.use(new LiffMockPlugin());

  const agent = navigator.userAgent;
  let coords = [0, 0];
  let profile = {};
  let errorMessage = "";

  async function init() {
    let mock = agent.includes("LIFF") ? false : true;
    return await liff.init({
      liffId: import.meta.env.VITE_LIFF_ID,
      mock,
    });
  }

  let promise = init();

  const setLiffMock = () => {
    liff.$mock.set((p) => ({
      ...p,
      getProfile: { displayName: "Cony *Mocked", userId: "1234" },
    }));
  };

  const closeWindow = () => {
    if (!liff.isInClient()) {
      window.alert(errorMessage);
    } else {
      liff.closeWindow();
    }
  };

  const sendMessage = () => {
    const today = moment().format("MM/DD");
    const message = [
      {
        type: "flex",
        altText: "this is a flex message",
        contents: {
          type: "bubble",
          size: "giga",
          body: {
            type: "box",
            layout: "vertical",
            contents: [
              {
                type: "box",
                layout: "horizontal",
                contents: [
                  {
                    type: "image",
                    url: "https://upload.wikimedia.org/wikipedia/commons/thumb/1/1b/Svelte_Logo.svg/199px-Svelte_Logo.svg.png",
                    margin: "none",
                    offsetTop: "md",
                    offsetBottom: "md",
                    offsetStart: "none",
                  },
                ],
                margin: "none",
              },
              {
                type: "text",
                text: `Today is ${today} `,
                align: "center",
                margin: "xl",
                size: "xl",
                color: "#eb336d",
              },
              {
                type: "separator",
                margin: "xxl",
              },
              {
                type: "box",
                layout: "vertical",
                contents: [
                  {
                    type: "text",
                    text: "Good luck with Svelte and LIFF",
                    wrap: true,
                    align: "center",
                  },
                ],
                borderColor: "#D3D3D3",
                borderWidth: "normal",
                cornerRadius: "sm",
                paddingAll: "lg",
                margin: "xxl",
              },
            ],
          },
          styles: {
            footer: {
              separator: true,
            },
          },
        },
      },
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
          window.alert("Error sending message: " + error);
        });
    }
  };

  onMount(async () => {
    if (!liff.isInClient()) {
      errorMessage =
        "The App is not opened in LINE, LIFF function will not work and will use Mocked User Info \n To open in line, use the QR below";
      // Run Login and Retrieve mock profile
      setLiffMock();
      liff.login();
      profile = await liff.getProfile();
      window.alert("not liff");
    } else {
      window.alert("in liff");
      profile = await liff.getProfile();
      window.alert("got profile");
    }

    window.alert("code here");
  });
</script>

<svelte:head>
  <title>LIFF Demo</title>
</svelte:head>
<Geolocation getPosition bind:coords />
<main>
  <h1>Liff Demo</h1>
  {#await promise}
    <p>LIFF init...</p>
  {:then}
    <p>LIFF init succeeded.</p>
    <div class="button">
      <button on:click={sendMessage}>Send Message</button>
      <button on:click={closeWindow}>Close</button>
    </div>

    <hr />
    <h3>User Info</h3>
    <ul>
      <li>
        <strong>User Name</strong>
        :<span>{profile.displayName}</span>
      </li>
      <li>
        <strong>GPS</strong>
        :<span>{JSON.stringify(coords)}</span>
      </li>
      <li>
        <strong>User Agent</strong>
        :<span>{agent}</span>
      </li>
    </ul>
    <h4 class="error">{errorMessage}</h4>
    <hr />
    <h3>LIFF Info</h3>
    <ul>
      <li>
        <strong>LIFF Browser</strong>
        :<span>{liff.isInClient()}</span>
      </li>
      <li>
        <strong>Login Status</strong>
        :<span>{liff.isLoggedIn()}</span>
      </li>
      <li>
        <strong>Language</strong>
        :<span>{liff.getLanguage()}</span>
      </li>
      <li>
        <strong>OS</strong>
        :<span>{liff.getOS()}</span>
      </li>
      <li>
        <strong>LIFF Ver</strong>
        :<span>{liff.getVersion()}</span>
      </li>
      <li>
        <strong>LINE Ver</strong>
        :<span>{liff.getLineVersion()}</span>
      </li>
    </ul>
    <hr />
    <h3>LIFF QR Code</h3>
    <img src="./liff-qr.png" alt="" />
  {:catch e}
    <p>LIFF init failed.</p>
    <p><code>{`${e}`}</code></p>
  {/await}
</main>

<style>
  main {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
  }
  .error {
    color: red;
  }
  ul {
    list-style-type: none;
    text-align: left;
    margin: 0;
    padding: 0;
  }
  li {
    padding: 5px;
    border: 1px solid #dddddd;
  }
  li span {
    margin: 5px;
  }
</style>
