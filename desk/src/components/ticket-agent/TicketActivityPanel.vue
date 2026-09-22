<template>
  <Tabs
    :modelValue="activeTab"
    :tabs="tabs"
    @update:modelValue="changeTabTo"
    size="md"
    class="flex-1 overflow-hidden [&_[role='tablist']]:px-5 [&_[role='tablist']]:py-1.5 [&_[role='tablist']]:flex-shrink-0 [&_[role='tabpanel'][data-state='active']]:flex-1 [&_[role='tabpanel'][data-state='active']]:flex [&_[role='tabpanel'][data-state='active']]:flex-col [&_[role='tabpanel'][data-state='active']]:overflow-auto [&_[role='tabpanel'][data-state='active']]:min-h-0"
  >
    <template #tab-panel="{ tab }">
      <TicketAnalyticsTab v-if="tab.value === 'analytics'" />
      <TicketTimeline
        v-else
        :ticket-id="String(ticket.doc?.name)"
        :tab="tab.value"
        :tab-label="tab.label"
        @email:reply="(e) => communicationAreaRef?.replyToEmail(e)"
      />
    </template>
  </Tabs>
  <!-- Comm Area -->
  <CommunicationArea
    ref="communicationAreaRef"
    :ticketId="String(ticket.doc?.name)"
    :to-emails="[ticket.doc?.raised_by]"
    :cc-emails="splitEmails(ticket.doc?.cc)"
    :bcc-emails="splitEmails(ticket.doc?.bcc)"
    :key="ticket.doc?.name"
    @update="reloadTicketFeed(String(ticket.doc?.name))"
  />
</template>

<script setup lang="ts">
import CommunicationArea from "@/components/CommunicationArea.vue";
import {
  ActivityIcon,
  CommentIcon,
  EmailIcon,
  PhoneIcon,
} from "@/components/icons";
import TicketAnalyticsTab from "@/components/ticket-agent/analytics/TicketAnalyticsTab.vue";
import { useActiveTabManager } from "@/composables/useActiveTabManager";
import { reloadTicketFeed } from "@/composables/useTicket";
import { useTelephonyStore } from "@/stores/telephony";
import { TabObject, TicketSymbol } from "@/types";
import { Tabs } from "frappe-ui";
import { storeToRefs } from "pinia";
import { computed, ComputedRef, inject, ref } from "vue";
import LucideChartNoAxesColumn from "~icons/lucide/chart-no-axes-column";
import TicketTimeline from "./timeline/TicketTimeline.vue";

const ticket = inject(TicketSymbol)!;

// Ticket-level CC/BCC (set on the main ticket form) are used as the default
// recipients on every reply, so agents don't have to retype them each time.
function splitEmails(value?: string): string[] {
  if (!value) return [];
  return value
    .split(",")
    .map((email) => email.trim())
    .filter(Boolean);
}

const communicationAreaRef = ref<InstanceType<typeof CommunicationArea> | null>(
  null
);
const telephonyStore = useTelephonyStore();
const { isCallingEnabled } = storeToRefs(telephonyStore);

const tabs: ComputedRef<TabObject[]> = computed(() => {
  const _tabs: TabObject[] = [
    {
      value: "activity",
      label: "Activity",
      iconLeft: ActivityIcon,
    },
    {
      value: "email",
      label: "Emails",
      iconLeft: EmailIcon,
    },
    {
      value: "comment",
      label: "Comments",
      iconLeft: CommentIcon,
    },
  ];

  if (isCallingEnabled.value) {
    _tabs.push({
      value: "call",
      label: "Calls",
      iconLeft: PhoneIcon,
    });
  }
  _tabs.push({
    value: "analytics",
    label: "Analytics",
    iconLeft: LucideChartNoAxesColumn,
  });
  return _tabs;
});

const { activeTab, changeTabTo } = useActiveTabManager(tabs);
</script>
