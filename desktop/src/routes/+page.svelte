<script context="module">
  export const prerender = true;
</script>

<script lang="ts">
  import { _ as C } from 'svelte-i18n'
  import Icon from '@iconify/svelte'
  import DashboardToolbar from '$lib/components/DashboardToolbar.svelte'
  import SiteFooter from '$lib/components/SiteFooter.svelte'
  import SiteThumbnail from '$lib/components/SiteThumbnail.svelte'
  import Modal, { show, hide } from '$lib/components/Modal.svelte'
  import sites from '../stores/sites'
  import serverSites, { connected } from '../stores/serverSites'
  import config from '../stores/config'
  import { goto } from '$app/navigation';
  import {setSitePreview, storeSite} from '$lib/actions'

  let loading
  function createSite() {
    show({
      id: 'SITE_CREATION',
      props: {
        onSuccess: (site) => {
          $sites = [
            ...$sites,
            {
              id: site.id,
              name: site.name,
              data: site,
              deployments: [],
              activeDeployment: null,
            },
          ]
          setSitePreview(site)
          storeSite(site)
          goto(site.id)
          hide()
        },
      },
    })
  }

  function connectToServer() {
    show({
      id: 'USER_SETTINGS',
      props: {
        tab: 1,
      },
    })
  }

  async function deleteSiteItem(siteID: string): Promise<any> {
    const confirm = window.confirm($C('dashboard.thumbnail.confirm_delete'))
    if (!confirm) return
    $sites = $sites.filter((site) => site.id !== siteID)
    window.primo?.data.deleteSite(siteID)
  }

  let siteBeingEdited

  let hoveredItem = null
</script>

<Modal />
<main class="primo-reset">
  <div class="container">
    <DashboardToolbar />
    <div class="sites-container">
      <ul class="sites" xyz="fade stagger stagger-1">
        {#each $sites as { data:site, preview }, i (site.id)}
          <li
            class="xyz-in"
            class:active={hoveredItem === i}
            class:inactive={hoveredItem !== null && hoveredItem !== i}
          >
            <a
              class="site-link"
              href={site.valid ? site.id : '/'}
              on:mouseenter={() => (hoveredItem = i)}
              on:mouseleave={() => (hoveredItem = null)}
            >
              <SiteThumbnail bind:valid={site.valid} {site} {preview} />
            </a>
            <div class="site-info">
              <div class="site-name">
                {#if siteBeingEdited === site.id}
                  <form
                    on:submit|preventDefault={() => (siteBeingEdited = null)}
                  >
                    <!-- svelte-ignore a11y-autofocus -->
                    <input
                      on:blur={() => (siteBeingEdited = null)}
                      class="reset-input"
                      type="text"
                      bind:value={site.name}
                      autofocus
                    />
                  </form>
                {:else}
                  <a
                    href={site.valid ? site.id : '/'}
                    on:mouseenter={() => (hoveredItem = i)}
                    on:mouseleave={() => (hoveredItem = null)}
                  >
                    <span>{site.name}</span>
                    {#if site.valid}
                      <svg
                        xmlns="http://www.w3.org/2000/svg"
                        viewBox="0 0 20 20"
                        fill="currentColor"
                        class="s-Uap-jPRb-uiE"
                        ><path
                          fill-rule="evenodd"
                          d="M7.293 14.707a1 1 0 010-1.414L10.586 10 7.293 6.707a1 1 0 011.414-1.414l4 4a1 1 0 010 1.414l-4 4a1 1 0 01-1.414 0z"
                          clip-rule="evenodd"
                          class="s-Uap-jPRb-uiE"
                        /></svg
                      >
                    {/if}
                  </a>
                {/if}
              </div>
              <span class="site-url">{site.id}</span>
              <div class="buttons">
                <button
                  class="delete-link"
                  on:click={() => (siteBeingEdited = site.id)}
                >
                  <Icon icon="bxs:edit" width="1rem" height="1rem" />
                  <span>{$C('dashboard.thumbnail.rename')}</span>
                </button>
                <button
                  class="delete-link"
                  on:click={() => deleteSiteItem(site.id)}
                >
                  <Icon icon="bxs:trash"  width="1rem" height="1rem" />
                  <span>{$C('dashboard.thumbnail.delete')}</span>
                </button>
              </div>
            </div>
          </li>
        {/each}
        <li
          class:inactive={hoveredItem !== true && hoveredItem !== null}
          class:active={hoveredItem === true}
          on:mouseenter={() => (hoveredItem = true)}
          on:mouseleave={() => (hoveredItem = null)}
        >
          <button class="create-site" on:click={createSite}>
            {#if loading}
              <!-- <Spinner /> -->
            {:else}
              <svg
                fill="currentColor"
                viewBox="0 0 20 20"
                xmlns="http://www.w3.org/2000/svg"
                ><path
                  fill-rule="evenodd"
                  d="M10 5a1 1 0 011 1v3h3a1 1 0 110 2h-3v3a1 1 0 11-2 0v-3H6a1 1 0 110-2h3V6a1 1 0 011-1z"
                  clip-rule="evenodd"
                /></svg
              >
            {/if}
            {$C('dashboard.create.heading')}
          </button>
        </li>
      </ul>
    </div>
    {#if $serverSites}
      <hr />
      <div class="sites-container">
        <header>
          <h2>{$C('dashboard.server.heading')}</h2>
          <a target="blank" rel="external" href={$config.serverConfig.url}
            >{$config.serverConfig.url}</a
          >
        </header>
        <ul class="sites" xyz="fade stagger stagger-1">
          {#each $serverSites as site, i (site.id)}
            <li class="xyz-in">
              <a href={site.id} class="site-link">
                <SiteThumbnail site={site.data} />
              </a>
              <div class="site-info">
                <div class="site-name">
                  {#if siteBeingEdited === site.id}
                    <form
                      on:submit|preventDefault={() => (siteBeingEdited = null)}
                    >
                      <input
                        on:blur={() => (siteBeingEdited = null)}
                        class="reset-input"
                        type="text"
                        bind:value={site.name}
                      />
                    </form>
                  {:else}
                    <a href={site.url}>{site.name}</a>
                    <button on:click={() => (siteBeingEdited = site.id)}>
                      <svg
                        xmlns="http://www.w3.org/2000/svg"
                        viewBox="0 0 20 20"
                        fill="currentColor"
                      >
                        <path
                          d="M17.414 2.586a2 2 0 00-2.828 0L7 10.172V13h2.828l7.586-7.586a2 2 0 000-2.828z"
                        />
                        <path
                          fill-rule="evenodd"
                          d="M2 6a2 2 0 012-2h4a1 1 0 010 2H4v10h10v-4a1 1 0 112 0v4a2 2 0 01-2 2H4a2 2 0 01-2-2V6z"
                          clip-rule="evenodd"
                        />
                      </svg>
                    </button>
                  {/if}
                </div>
                <span class="site-url">{site.id} </span>
                <div class="buttons">
                  <button
                    class="delete-link"
                    on:click={() => deleteSiteItem(site.id)}
                  >
                    <Icon icon="bxs:trash"  width="1rem" height="1rem" />
                    <span>{$C('dashboard.thumbnail.delete')}</span>
                  </button>
                </div>
              </div>
            </li>
          {:else}
            <span class="info">{$C('dashboard.server.empty')}</span>
          {/each}
          {#if !$connected}
            <li>
              <button class="create-site" on:click={connectToServer}>
                <span>{$C('dashboard.thumbnail.connect')}</span>
              </button>
            </li>
          {/if}
        </ul>
      </div>
    {/if}
    <SiteFooter />
  </div>
</main>

<style lang="postcss">
  main {
    background-color: var(--primo-color-black);
    min-height: 100vh;
    position: relative;
    z-index: 0;
    padding-top: 1rem;

    .container {
      display: flex;
      flex-direction: column;
      border-radius: var(--primo-border-radius);
      margin: 0 auto;
      padding: 2rem;
      min-height: 100vh;
    }

    hr {
      margin: 2rem 0;
      border-color: var(--color-gray-9);
    }

    .sites-container {
      flex: 1;
      display: grid;
      gap: 1rem;

      span.info {
        padding: 1rem;
        color: white;
        background: var(--color-gray-9);
      }

      header {
        h2 {
          color: white;
        }
        a {
          text-decoration: underline;
          color: var(--color-gray-4);
        }
      }

      ul.sites {
        display: grid;
        place-content: flex-start;
        place-items: flex-start normal;
        gap: 1rem;

        li {
          height: 100%;
          background: var(--color-gray-9);
          border-radius: var(--primo-border-radius);
          overflow: hidden;
          font-size: var(--font-size-4);
          box-shadow: var(--box-shadow-md);
          display: flex;
          flex-direction: column;
          justify-content: flex-start;
          transition: opacity 0.1s, filter 0.1s;

          &.active {
            filter: brightness(1.1);
          }

          &.inactive {
            opacity: 0.5;
          }

          .site-link {
            background: var(--color-gray-8);
            transition: opacity 0.1s;
          }

          .site-info {
            flex: 1;
            color: var(--color-gray-1);
            display: flex;
            flex-direction: column;
            justify-content: flex-start;
            padding: 1.5rem;

            .site-name {
              display: flex;

              a {
                display: flex;

                &:hover {
                  color: var(--primo-color-primogreen);
                }

                svg {
                  width: 1.5rem;
                  margin-top: 3px;
                }
              }

              a,
              input {
                text-align: left;
                font-size: var(--font-size-4);
                font-weight: 600;
              }

              button {
                border-radius: var(--primo-border-radius);
                padding: 0 0.5rem;

                &:hover {
                  color: var(--primo-color-primogreen);
                }

                svg {
                  width: 1rem;
                  height: 1rem;
                }
              }
            }

            .site-url {
              grid-column: 1;
              margin-bottom: 0.5rem;
              font-size: var(--font-size-1);
              color: var(--color-gray-4);
            }

            .buttons {
              grid-column: 1;
              display: flex;
              align-items: center;
              color: var(--color-gray-3);
              margin-top: 0.5rem;

              button {
                margin-right: 0.75rem;
              }
            }
          }
        }

        button.create-site {
          padding: 3rem;
          display: flex;
          flex-direction: column;
          align-items: center;
          justify-content: center;
          width: 100%;
          height: 100%;
          background: var(--primo-color-black);
          font-weight: 600;
          color: var(--color-gray-2);
          border-radius: var(--primo-border-radius);
          border: 2px solid var(--primo-color-primogreen);

          svg {
            border-radius: 50%;
            transition: background 0.1s;
            height: 2rem;
            width: 2rem;
          }

          &:hover svg {
            color: var(--primo-color-primogreen);
          }
        }
      }
    }
  }

  .delete-link {
    display: flex;
    align-items: center;
    font-size: 0.75rem;
    color: var(--color-gray-2);
    text-decoration: underline;
    span {
      padding-left: 0.25rem;
    }
    &:hover {
      color: var(--primo-color-primogreen);
    }
  }

  button {
    transition: color 0.1s, background-color 0.1s;
    &:focus {
      outline: 2px solid var(--primo-color-primogreen);
    }
  }

  .reset-input {
    border: none;
    background-image: none;
    background-color: transparent;
    box-shadow: none;
    background: transparent;
    border: 0;
    padding: 0;
    margin: 0;

    &:focus {
      outline: 0;
    }
  }

  @media (min-width: 600px) {
    main .sites-container ul.sites {
      grid-template-columns: 1fr 1fr;
    }
  }

  @media (min-width: 900px) {
    main .sites-container ul.sites {
      grid-template-columns: 1fr 1fr 1fr;
    }
  }

  @media (min-width: 1200px) {
    main .sites-container ul.sites {
      grid-template-columns: 1fr 1fr 1fr 1fr;
    }
  }
</style>
