<script setup lang="ts">
import { ref, onMounted, computed } from 'vue'
import { useI18n } from 'vue-i18n'
import { adminAPI } from '@/api/admin'
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card'
import { Badge } from '@/components/ui/badge'
import { Button } from '@/components/ui/button'
import { formatDate } from '@/utils/format'
import { Wifi, WifiOff, RefreshCw } from 'lucide-vue-next'

const { t } = useI18n()

const loading = ref(false)
const runtimeStatus = ref<Record<string, unknown> | null>(null)

const isConnected = computed(() => {
  if (!runtimeStatus.value) return false
  return runtimeStatus.value.connected === true
})

const fetchRuntimeStatus = async () => {
  loading.value = true
  try {
    const res = await adminAPI.getTelegramBotRuntimeStatus()
    const data = res.data?.data
    runtimeStatus.value = data && typeof data === 'object' ? (data as Record<string, unknown>) : null
  } catch {
    runtimeStatus.value = null
  } finally {
    loading.value = false
  }
}

const formatRuntimeDate = (value: unknown) => {
  if (typeof value !== 'string' || !value) return '-'
  return formatDate(value) || '-'
}

const formatWebhookStatus = (value: unknown) => {
  if (typeof value !== 'string' || !value) return '-'
  const normalized = value.trim().toLowerCase()
  if (['active', 'enabled', 'connected', 'ok'].includes(normalized)) {
    return t('telegramBot.status.webhookStatusActive')
  }
  if (['inactive', 'disabled', 'disconnected'].includes(normalized)) {
    return t('telegramBot.status.webhookStatusInactive')
  }
  return value
}

onMounted(() => {
  fetchRuntimeStatus()
})
</script>

<template>
  <div class="space-y-6">
    <div class="flex items-center justify-between">
      <div>
        <h2 class="text-2xl font-bold tracking-tight">{{ t('telegramBot.status.title') }}</h2>
        <p class="text-muted-foreground">{{ t('telegramBot.status.subtitle') }}</p>
      </div>
      <Button variant="outline" size="sm" :disabled="loading" @click="fetchRuntimeStatus">
        <RefreshCw class="h-4 w-4 mr-2" :class="{ 'animate-spin': loading }" />
        {{ t('telegramBot.overview.refreshStatus') }}
      </Button>
    </div>

    <Card>
      <CardHeader>
        <div class="flex items-center justify-between">
          <div>
            <CardTitle>{{ t('telegramBot.status.connectionStatus') }}</CardTitle>
            <CardDescription>{{ t('telegramBot.status.connectionStatusDesc') }}</CardDescription>
          </div>
          <Badge :variant="isConnected ? 'default' : 'secondary'">
            <component :is="isConnected ? Wifi : WifiOff" class="h-3 w-3 mr-1" />
            {{ isConnected ? t('telegramBot.overview.connected') : t('telegramBot.overview.notConnected') }}
          </Badge>
        </div>
      </CardHeader>
      <CardContent>
        <div v-if="runtimeStatus" class="space-y-4">
          <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
            <div class="rounded-lg border p-4">
              <p class="text-sm text-muted-foreground mb-1">{{ t('telegramBot.status.connected') }}</p>
              <p class="text-lg font-semibold">
                <Badge :variant="isConnected ? 'default' : 'destructive'">
                  {{ isConnected ? t('telegramBot.overview.connected') : t('telegramBot.overview.notConnected') }}
                </Badge>
              </p>
            </div>
            <div class="rounded-lg border p-4">
              <p class="text-sm text-muted-foreground mb-1">{{ t('telegramBot.status.botVersion') }}</p>
              <p class="text-lg font-semibold">{{ runtimeStatus.bot_version || '-' }}</p>
            </div>
            <div class="rounded-lg border p-4">
              <p class="text-sm text-muted-foreground mb-1">{{ t('telegramBot.status.webhookStatus') }}</p>
              <p class="text-lg font-semibold">{{ formatWebhookStatus(runtimeStatus.webhook_status) }}</p>
            </div>
            <div class="rounded-lg border p-4">
              <p class="text-sm text-muted-foreground mb-1">{{ t('telegramBot.status.configVersion') }}</p>
              <p class="text-lg font-semibold">{{ runtimeStatus.config_version ?? '-' }}</p>
            </div>
            <div class="rounded-lg border p-4">
              <p class="text-sm text-muted-foreground mb-1">{{ t('telegramBot.status.lastSeenAt') }}</p>
              <p class="text-lg font-semibold">{{ formatRuntimeDate(runtimeStatus.last_seen_at) }}</p>
            </div>
            <div class="rounded-lg border p-4">
              <p class="text-sm text-muted-foreground mb-1">{{ t('telegramBot.status.lastConfigSyncAt') }}</p>
              <p class="text-lg font-semibold">{{ formatRuntimeDate(runtimeStatus.last_config_sync_at) }}</p>
            </div>
          </div>
        </div>
        <div v-else class="rounded-lg border border-dashed p-6 text-center">
          <WifiOff class="h-10 w-10 mx-auto text-muted-foreground mb-3" />
          <p class="text-sm text-muted-foreground">{{ t('telegramBot.overview.notConnectedHint') }}</p>
        </div>
      </CardContent>
    </Card>
  </div>
</template>
