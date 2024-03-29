<script>
  import axios from "axios";
  import { intervalToDuration } from "date-fns";
  import { onMount } from "svelte";
  import About from "./aboutCountdown.svelte";

  let items;

  async function getExpiringENS() {
    try {
      const result = await axios.post(
        "https://api.thegraph.com/subgraphs/name/ensdomains/ens",
        {
          query: `
					query getRegistrantFromSubgraph {
						registrations(
								first: 50,
								orderBy: expiryDate,
								orderDirection: asc,
								where: {
									expiryDate_gte: ${Math.round(new Date().getTime() / 1000)}
								}
						) {
								domain {
									id
									name
								}
								expiryDate
							}
						}
					`,
        }
      );

      const itemsWithCountdown = result.data.data.registrations.map((item) => {
        return {
          ...item,
          countdown: intervalToDuration({
            start: item.expiryDate * 1000,
            end: Date.now(),
          }),
        };
      });

      items = itemsWithCountdown;
    } catch (error) {
      console.error(error);
    }
  }

  onMount(() => {
    setInterval(() => {
      items = items.map((item) => {
        return {
          ...item,
          countdown: intervalToDuration({
            start: item.expiryDate * 1000,
            end: Date.now(),
          }),
        };
      });
    }, 1000);
  });
</script>

<div>
  <About />
  <section id="about">
    <div class="overflow-x-auto">
      <table class="table w-full table-zebra">
        <thead>
          <tr>
            <th />
            <th>ENS Domain</th>
            <th>Countdown</th>
            <th>Expiration Date</th>
            <th>Useful Links</th>
          </tr>
        </thead>
        <tbody>
          {#await getExpiringENS()}
            <p>Loading...</p>
          {:then expiringENS}
            {#each items as item, index}
              <tr>
                <th>{index + 1}</th>
                <td>{item.domain.name}</td>
                <td>
                  <div
                    class="grid grid-flow-col gap-5 place-items-end auto-cols-max"
                  >
                    <span class="font-mono text-2xl countdown">
                      <span style="--value:{item.countdown.hours || 0};" />h
                      <span style="--value:{item.countdown.minutes || 0};" />m
                      <span style="--value:{item.countdown.seconds || 0};" />s
                    </span>
                  </div>
                </td>
                <td>{new Date(item.expiryDate * 1000).toLocaleString()}</td>
                <td>
                  <a
                  id="link"
                    class="link"
                    href="https://app.ens.domains/name/{item.domain
                      .name}/details">
                         <img
                         id="image"
                          src="https://www.pngkit.com/png/full/152-1526200_ens-logo-ethereum-name-service.png"
                          class="rounded-xl"
                        />
                      </a
                  >
                  <a
                  id="link"
                    class="link"
                    href="https://opensea.io/collection/ens?search[query]={item
                      .domain.name}"> <img
                         id="image"
                          src="https://user-images.githubusercontent.com/35243/140804979-0ef11e0d-d527-43c1-93cb-0f48d1aec542.png"
                          class="rounded-xl"
                        /></a
                  >
                </td>
              </tr>
            {/each}
          {/await}
        </tbody>
      </table>
    </div>
  </section>
</div>

<style lang="scss">
  @import "./scss/breakpoints.scss";

  #about {
    position: relative;
    display: flex;
    grid-template-columns: 500px 250px;
    align-items: center;
    justify-content: center;
    gap: 15px;
    padding: 80px 0;

    @include for-phone-only {
      grid-template-columns: 1fr;
      justify-items: center;
    }

    .info {
      display: flex;
      flex-direction: column;
      gap: 10px;
      h2 {
        @include for-phone-only {
          text-align: center;
        }
      }

      p {
        @include for-phone-only {
          text-align: justify;
        }
      }
    }
  }

  @keyframes spin {
    from {
      transform: rotate(0turn);
    }
    to {
      transform: rotate(1turn);
    }
  }

  #image {
    height: 37px;
    width: auto;
  margin-left: auto;
  margin-right: auto;
  display: inline
  }
  #link {
    display: inline;
}

</style>
