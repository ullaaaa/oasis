<script lang="ts">
  import { t } from "svelte-i18n";
  import Modal from "../components/Modal.svelte";
  import Button from "../components/Button.svelte";
  import type {
    ILink,
    IUpdateAppNeedRespose,
    IUpdateAppInfo,
  } from "../utils/types";
  import { setNotification, siteStore } from "../utils/store";
  import * as api from "../utils/api";
  import { compareVersion } from "../utils/util";
  import * as constants from "../assets/constants.json";

  export let onClose: () => void;
  export let setUpdateInfo: (info: IUpdateAppInfo) => void;
  let isLoading = false;
  let repo: ILink = constants.repo;
  let links: Array<ILink> = constants.links;
  let version = ($siteStore && $siteStore.version) || "0.1";

  const checkUpdate = async () => {
    if (!$siteStore) {
      return;
    }

    isLoading = true;

    try {
      const response: IUpdateAppNeedRespose = await api.get("/api/sys/update");
      const updateInfo: IUpdateAppInfo = await api.get(response.url);
      if (compareVersion(updateInfo.version, $siteStore.version) > 0) {
        setUpdateInfo(updateInfo);
      } else {
        setNotification("success", $t("message.success.app_up_to_date"));
      }
    } catch (e) {
      console.error(e);
      setNotification("error", $t("message.error.get_update_info_error"));
    }

    isLoading = false;
  };
</script>

<Modal {onClose} title={$t("modal.about.title")}>
  <div class="p-4 text-lg">
    <div>
      <span>{$t("modal.about.version")}: </span>
      <span class="ml-2 px-1">{version}</span>
    </div>
    {#if repo}
      <div>
        {$t("modal.about.repo")}:
        <a
          href={repo.url}
          target="_blank"
          class="px-1 ml-2 rounded-sm hover:bg-blue-400 hover:text-white break-all"
        >
          {repo.name}
        </a>
      </div>
    {/if}
    <div class="mt-4">{$t("modal.about.thanks")} ❤</div>
    <div>
      {#each links as link}
        <a
          href={link.url}
          target="_blank"
          class="px-1 rounded-sm hover:bg-blue-400 hover:text-white"
          >{link.name}</a
        >
        |{" "}
      {/each}
      ...
    </div>
  </div>

  <div class="w-full p-4 flex flex-row justify-end">
    <Button
      onClick={checkUpdate}
      color="blue"
      value={isLoading
        ? $t("button.check_updating")
        : $t("button.check_update")}
      className="mr-4"
      disabled={isLoading}
    />
    <Button onClick={onClose} value={$t("button.close")} />
  </div>
</Modal>
