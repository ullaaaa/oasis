<script lang="ts">
  import { onDestroy } from "svelte";
  import { t } from "svelte-i18n";
  import { notificationStore } from "../utils/store";
  import { EIconType, EIconColor } from "../utils/types";
  import type { INotification } from "../utils/types";
  import Icon from "../components/Icon.svelte";

  let notifications: INotification[] = [];
  let isShowNotification = false;
  let isManulClosed = false;
  let currentNotification: INotification;
  let type: "success" | "error";
  let msg = "";
  let timeout: NodeJS.Timeout;

  const unsubscribe = notificationStore.subscribe((value) => {
    if (value) {
      if (currentNotification) {
        notifications.push(value);
      } else {
        currentNotification = value;
      }
      notificationStore.set(null);
    }
  });

  $: if (isManulClosed && notifications.length > 0) {
    timeout = setTimeout(() => {}, 500);
    isManulClosed = false;
    showNotification();
  }

  $: if (currentNotification) {
    isShowNotification = true;
    showNotification();
  }

  $: if (currentNotification) {
    type = currentNotification.type;
    msg = currentNotification.msg;
  }

  const buildStyle = () => {
    let style =
      "flex flex-row justify-between bg-gray-50 shadow rounded border-l-4 items-center p-2 notification";

    switch (currentNotification.type) {
      case "success":
        style += " border-green-400";
        break;
      case "error":
        style += " border-red-400";
        break;
      default:
        break;
    }
    return style;
  };

  const showNotification = () => {
    timeout = setTimeout(() => {
      if (notifications.length > 0) {
        currentNotification = notifications.shift();
      } else {
        currentNotification = null;
        isShowNotification = false;
      }
    }, 4000);
  };

  const closeNotification = () => {
    if (timeout) {
      clearTimeout(timeout);
    }

    currentNotification = null;
    isShowNotification = false;
    isManulClosed = true;
  };

  onDestroy(() => {
    unsubscribe();
  });
</script>

{#if isShowNotification}
  <div class="absolute right-6 top-20 w-60 lg:w-72 overflow-x-hidden z-50">
    <div class={buildStyle()}>
      <div class="flex flex-row items-center">
        <div class="mr-2">
          <Icon
            type={type === "success" ? EIconType.success : EIconType.error}
            color={type === "success" ? EIconColor.green : EIconColor.red}
          />
        </div>
        <div class="flex flex-col">
          <div class="text-gray-700 font-bold">
            {$t("message.type." + type)}
          </div>
          <div class="text-sm">{msg}</div>
        </div>
      </div>
      <div class="text-3xl cursor-pointer" on:click={closeNotification}>
        &times;
      </div>
    </div>
  </div>
{/if}

<style>
  .notification {
    animation: notify 4s ease-in-out infinite;
  }

  @keyframes notify {
    0% {
      transform: translateX(120%);
    }
    15% {
      transform: translateX(0);
    }
    90% {
      transform: translateX(0);
    }
    100% {
      transform: translateX(120%);
    }
  }
</style>
