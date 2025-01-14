<script>
  import { _ as C } from 'svelte-i18n'
  import * as Mousetrap from 'mousetrap';
  import { createEventDispatcher, onMount } from 'svelte';
  import { browser } from '$app/environment';

  const dispatch = createEventDispatcher();

  import Toolbar from './Toolbar.svelte';
  import ToolbarButton from './ToolbarButton.svelte';
  import Doc from './Doc.svelte';

  import { showingIDE } from '../../stores/app';
  import { fields as siteFields, timeline } from '../../stores/data/draft';
  import { fields as pageFields, sections } from '../../stores/app/activePage';
  import { saving, saved, loadingSite } from '../../stores/app/misc';
  import modal from '../../stores/app/modal';
  import { undoSiteChange, redoSiteChange, changeLocale } from '../../stores/actions';

  $: pageEmpty = $sections && $sections.length <= 1 && $sections.length > 0 && $sections[0]['type'] === 'options';

  // setup key-bindings
  onMount(() => {
    Mousetrap.bind(['mod+s'], (e) => {
      e.preventDefault();
      savePage();
    });

    Mousetrap.bind(['mod+l'], (e) => {
      e.preventDefault();
      changeLocale()
    });

    Mousetrap.bind(['mod+d'], (e) => {
      e.preventDefault();
      showingIDE.set(!$showingIDE)
    });
  });

  $: hasFields = $siteFields ? [...$siteFields, ...$pageFields].length > 0 : false

  $: editorButtons = [
    [
      {
        id: 'pages',
        title: $C('Pages'),
        label: $C('Pages'),
        icon: 'fa-solid:th-large',
        onclick: () => modal.show('SITE_PAGES', {}, { hideLocaleSelector: true }),
        showSwitch: false
      },
    ],
    hasFields ? [
      {
        icon: 'bxs:edit',
        title: $C('Content'),
        label: $C('Content'),
        onclick: () => modal.show('FIELDS', {}, {
          showSwitch: true
        }),
      },
    ] : [],
  ];

  const developerButtons = [
    [
      {
        id: 'toolbar--pages',
        label: $C('Pages'),
        title: $C('Pages'),
        icon: 'fa-solid:th-large',
        onclick: () => modal.show('SITE_PAGES', {}, { hideLocaleSelector: true }),
        showSwitch: false
      },
    ],
    [
      {
        id: 'toolbar--components',
        title: $C('Components'),
        label: $C('Components'),
        icon: 'fa6-solid:clone',
        onclick: () => modal.show('SYMBOL_LIBRARY'),
      },
    ],
    [
      {
        id: 'toolbar--html',
        title: 'HTML',
        label: 'HTML',
        icon: 'icomoon-free:html-five2',
        onclick: () => modal.show('WRAPPER'),
      },
      {
        id: 'toolbar--css',
        title: 'CSS',
        label: 'CSS',
        icon: 'ci:css3',
        onclick: () => modal.show('STYLES'),
      },
      {
        id: 'toolbar--fields',
        title: $C('Fields'),
        label: $C('Fields'),
        icon: 'bxs:edit',
        onclick: () => modal.show('FIELDS', {}, {
          showSwitch: true
        }),
      },
    ],
    // [
    //   {
    //     id: 'toolbar--preview',
    //     title: 'Preview',
    //     icon: 'window-restore',
    //     onclick: async () => {
    //       // window.open('http://localhost:3333', '_blank');
    //       // var wnd = window.open('about:blank', '', '_blank');
    //       // wnd.document.write(iframePreview);
    //       const w = window.open(
    //         'data:text/html;charset=utf-8,' + currentPagePreview,
    //         '',
    //         '_blank'
    //       );
    //       const preview = await buildStaticPage({
    //         page: $currentPage,
    //         site: $site,
    //       });
    //       console.log({ preview });
    //       console.log({ w });
    //       w.postMessage({ html: preview });
    //       // setTimeout(() => {
    //       //   w.postMessage({ html: preview });
    //       // }, 0);
    //     },
    //   },
    // ],
  ];

  function savePage() {
    dispatch('save');
  }

  $: toolbarButtons = $showingIDE ? developerButtons : editorButtons;

  // Show 'are you sure you want to leave prompt' when closing window
  $: if (browser && !$saved && window.location.hostname !== 'localhost') {
    window.onbeforeunload = function (e) {
      e.returnValue = '';
    };
  } else if (browser) {
    window.onbeforeunload = function (e) {
      delete e['returnValue'];
    };
  }

  // Add top margin to page since toolbar is fixed
  let toolbar;
  let page;
  $: if (toolbar && page) {
    setTimeout(() => {
      page.style.borderTop = `${
        toolbar.clientHeight
      }px solid var(--primo-color-black)`;
    }, 0)
  }
  
</script>

<Toolbar
  bind:element={toolbar}
  on:signOut
  buttons={$loadingSite ? [] : toolbarButtons}
  on:toggleView={() => showingIDE.set(!$showingIDE)}
>
  {#if !$timeline.first}
    <ToolbarButton
      id="undo"
      title="Undo"
      icon="undo"
      on:click={undoSiteChange}
    />
  {/if}
  {#if !$timeline.last}
    <ToolbarButton
      id="redo"
      title="Redo"
      icon="redo"
      on:click={redoSiteChange}
    />
  {/if}
  <ToolbarButton
    id="save"
    title="Save"
    icon="save"
    key="s"
    loading={$saving}
    on:click={savePage}
    disabled={$saved}
  />
  <ToolbarButton
    type="primo"
    title={pageEmpty ? 'Add a content block or component to your page to publish it' : 'Publish'}
    label="Publish"
    icon="globe"
    style="margin-left:0.25rem;"
    active={false}
    on:click={() => modal.show('BUILD')}
    disabled={pageEmpty}
  />
</Toolbar>

<Doc bind:element={page} on:save />
