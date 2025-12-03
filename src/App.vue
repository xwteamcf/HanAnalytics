<template>
  <section class="han_analytics" :class="currentTheme">
    <header>
      <div class="main">
        <div class="logo">
          <img src="./assets/favicon.ico">
          <span>Analytics</span>
        </div>
        <h2 v-html="dateInfo || '您当前正在访问网站「Analysis」'"></h2>
      </div>
    </header>
    <main>
      <header>
        <Alert>
          <AlertDescription>
            <p>· Han Analytics 是一个简单的网络分析跟踪器和仪表板，托管在被称为赛博菩萨的 Cloudflare 上,无成本稳定运行,每天可达10万次免费统计。</p>
            <p>· 域名、服务器、数据库 通通都不用! 托管在 Cloudflare Pages 上即可快速部署网站分析仪表板。</p>
            <p style="font-weight: bold;">· 开源地址: <a class="git-link" href="https://github.com/uxiaohan/HanAnalytics"
                target="_blank">Han-Analytics</a>
            </p>
          </AlertDescription>
        </Alert>
      </header>

      <section class="main">
        <div class="pb-5 grid md:grid-cols-2 sm:grid-cols-1 items-start">
          <div class="flex gap-[16px] pb-6">
            <div class="w-3/6">
              <Select :disabled="siteList.length < 1 || getDatasStatus" v-model="siteValue"
                @update:model-value="getDatas">
                <SelectTrigger class="w-[218px]">
                  <SelectValue placeholder="选择站点" />
                </SelectTrigger>
                <SelectContent>
                  <SelectGroup>
                    <SelectLabel>选择站点</SelectLabel>
                    <SelectItem :value="i" v-for="i in siteList" :key="i">{{ i }}</SelectItem>
                  </SelectGroup>
                </SelectContent>
              </Select>
            </div>
            <div class="w-3/6">
              <Select :disabled="siteList.length < 1 || getDatasStatus" v-model="timeValue"
                @update:model-value="getDatas">
                <SelectTrigger class="w-[218px]">
                  <SelectValue placeholder="选择周期" />
                </SelectTrigger>
                <SelectContent>
                  <SelectGroup>
                    <SelectLabel>选择周期</SelectLabel>
                    <SelectItem :value="i.value" v-for="i in timeList" :key="i.name">{{ i.name }}</SelectItem>
                  </SelectGroup>
                </SelectContent>
              </Select>
            </div>
          </div>
          <div
            class="flex justify-end text-center md:text-right line-clamp-1 [&>.views-item]:flex [&>.views-item]:flex-col [&>.views-item]:items-center md:[&>.views-item]:items-end [&>.views-item]:gap-4 [&>.views-item>span]:text-sm [&>.views-item>p]:text-3xl [&>.views-item>p]:line-clamp-1 [&>.views-item>p]:w-full">
            <div class="views-item w-full overflow-hidden">
              <span>浏览量</span>
              <div class="space-y-2 w-[50%]" v-if="resData.visit.views === undefined">
                <Skeleton class="h-4  w-[50%] ml-auto" />
                <Skeleton class="h-4" />
              </div>
              <p v-else>{{ resData.visit.views }}</p>
            </div>
            <div class="views-item w-full overflow-hidden">
              <span>访客数</span>
              <div class="space-y-2 w-[50%]" v-if="resData.visit.visitor === undefined">
                <Skeleton class="h-4  w-[50%] ml-auto" />
                <Skeleton class="h-4" />
              </div>
              <p v-else>{{ resData.visit.visitor }}</p>
            </div>
            <div class="views-item w-full overflow-hidden">
              <span>访问次数</span>
              <div class="space-y-2 w-[50%]" v-if="resData.visit.visit === undefined">
                <Skeleton class="h-4  w-[50%] ml-auto" />
                <Skeleton class="h-4" />
              </div>
              <p v-else>{{ resData.visit.visit }}</p>
            </div>
          </div>
        </div>

        <div ref="echartsDOM" class="data-view"></div>

        <div class="pt-20 grid md:grid-cols-2 sm:grid-cols-1 gap-[16px]">
          <Card class="box-border flex flex-col w-full h-[460px] overflow-hidden">
            <CardHeader>
              <CardTitle>访问页面</CardTitle>
            </CardHeader>
            <CardContent class="box-border pt-0 w-full h-full overflow-hidden">
              <ScrollArea class="box-border p-2 pt-0 h-full w-full pages-list" v-if="resData.path != undefined">
                <p class="page-item" v-for="(i, idx) in resData.path" :key="idx">
                  <span class="line-clamp-1">{{ i.name }}</span>
                  <span class="line-clamp-1">{{ i.value }}</span>
                  <em>{{ i.per }}<i :style="{ width: i.per }"></i></em>
                </p>
              </ScrollArea>
              <div class="space-y-4 pt-8 w-full" v-else>
                <Skeleton class="h-6 w-60" />
                <Skeleton class="h-6 w-80" />
                <Skeleton class="h-6 w-100" />
                <Skeleton class="h-6 w-60" />
                <Skeleton class="h-6 w-80" />
                <Skeleton class="h-6 w-100" />
                <Skeleton class="h-6 w-80" />
                <Skeleton class="h-6 w-full" />
              </div>
            </CardContent>
          </Card>

          <Card class="box-border flex flex-col w-full h-[460px] overflow-hidden">
            <CardHeader>
              <CardTitle>来源网站</CardTitle>
            </CardHeader>
            <CardContent class="box-border pt-0 w-full h-full overflow-hidden">
              <ScrollArea class="box-border p-2 pt-0 h-full w-full pages-list" v-if="resData.referrer != undefined">
                <p class="page-item" v-for="(i, idx) in resData.referrer" :key="idx">
                  <img v-if="i.name" :src="getIconUrl(i.name)">
                  <a :href="i.name" target="_blank" rel="noopener noreferrer" class="line-clamp-1 cursor-pointer">
                    {{ i.name || '(None)' }}
                  </a>
                  <span class="line-clamp-1">{{ i.value }}</span>
                  <em>{{ i.per }}<i :style="{ width: i.per }"></i></em>
                </p>
              </ScrollArea>
              <div class="space-y-4 pt-8 w-full" v-else>
                <Skeleton class="h-6 w-60" />
                <Skeleton class="h-6 w-80" />
                <Skeleton class="h-6 w-100" />
                <Skeleton class="h-6 w-60" />
                <Skeleton class="h-6 w-80" />
                <Skeleton class="h-6 w-100" />
                <Skeleton class="h-6 w-80" />
                <Skeleton class="h-6 w-full" />
              </div>
            </CardContent>
          </Card>
        </div>

        <div class="pt-6 grid xl:grid-cols-3 gap-[16px] md:grid-cols-2 sm:grid-cols-1">
          <Card class="box-border flex flex-col w-full h-[460px] overflow-hidden">
            <CardHeader>
              <CardTitle>浏览器</CardTitle>
            </CardHeader>
            <CardContent class="box-border pt-0 w-full h-full overflow-hidden">
              <ScrollArea class="box-border p-2 pt-0 h-full w-full pages-list" v-if="resData.soft != undefined">
                <p class="page-item" v-for="(i, idx) in resData.soft" :key="idx">
                  <span class="line-clamp-1">{{ i.name }}</span>
                  <span class="line-clamp-1">{{ i.value }}</span>
                  <em>{{ i.per }}<i :style="{ width: i.per }"></i></em>
                </p>
              </ScrollArea>
              <div class="space-y-4 pt-8 w-full" v-else>
                <Skeleton class="h-6 w-60" />
                <Skeleton class="h-6 w-80" />
                <Skeleton class="h-6 w-100" />
                <Skeleton class="h-6 w-60" />
                <Skeleton class="h-6 w-80" />
                <Skeleton class="h-6 w-100" />
                <Skeleton class="h-6 w-80" />
                <Skeleton class="h-6 w-full" />
              </div>
            </CardContent>
          </Card>

          <Card class="box-border flex flex-col w-full h-[460px] overflow-hidden">
            <CardHeader>
              <CardTitle>操作系统</CardTitle>
            </CardHeader>
            <CardContent class="box-border pt-0 w-full h-full overflow-hidden">
              <ScrollArea class="box-border p-2 pt-0 h-full w-full pages-list" v-if="resData.os != undefined">
                <p class="page-item" v-for="(i, idx) in resData.os" :key="idx">
                  <img class="os" :src="getIcon(i.name)">
                  <span class="line-clamp-1">{{ i.name }}</span>
                  <span class="line-clamp-1">{{ i.value }}</span>
                  <em>{{ i.per }}<i :style="{ width: i.per }"></i></em>
                </p>
              </ScrollArea>
              <div class="space-y-4 pt-8 w-full" v-else>
                <Skeleton class="h-6 w-60" />
                <Skeleton class="h-6 w-80" />
                <Skeleton class="h-6 w-100" />
                <Skeleton class="h-6 w-60" />
                <Skeleton class="h-6 w-80" />
                <Skeleton class="h-6 w-100" />
                <Skeleton class="h-6 w-80" />
                <Skeleton class="h-6 w-full" />
              </div>
            </CardContent>
          </Card>


          <Card class="box-border flex flex-col w-full h-[460px] overflow-hidden">
            <CardHeader>
              <CardTitle>访问地区</CardTitle>
            </CardHeader>
            <CardContent class="box-border pt-0 w-full h-full overflow-hidden">
              <ScrollArea class="box-border p-2 pt-0 h-full w-full pages-list" v-if="resData.area != undefined">
                <p class="page-item" v-for="(i, idx) in resData.area" :key="idx">
                  <img :src="getIcon(i.name)">
                  <span class="line-clamp-1">{{ i.code }}</span>
                  <span class="line-clamp-1">{{ i.value }}</span>
                  <em>{{ i.per }}<i :style="{ width: i.per }"></i></em>
                </p>
              </ScrollArea>
              <div class="space-y-4 pt-8 w-full" v-else>
                <Skeleton class="h-6 w-60" />
                <Skeleton class="h-6 w-80" />
                <Skeleton class="h-6 w-100" />
                <Skeleton class="h-6 w-60" />
                <Skeleton class="h-6 w-80" />
                <Skeleton class="h-6 w-100" />
                <Skeleton class="h-6 w-80" />
                <Skeleton class="h-6 w-full" />
              </div>
            </CardContent>
          </Card>
        </div>
      </section>
    </main>
    <footer>
      <p><img src="./assets/svg/ing.svg"></p>
      <p>
        <a href="https://www.xwteam.cn" target="_blank" rel="noopener noreferrer"><img
            src="./assets/svg/blog.svg"></a>
        <a href="https://api.xwteam.cn" target="_blank"
          rel="noopener noreferrer"><img src="./assets/svg/api.svg"></a>
        <a href="https://free.xwteam.cn" target="_blank" rel="noopener noreferrer"><img src="./assets/svg/freeapi.svg"></a>
        <a href="https://analysis.xwteam.com/" target="_blank" rel="noopener noreferrer"><img src="./assets/svg/analysis.svg"></a>
      </p>
      <p>
        <a href="https://pages.cloudflare.com" target="_blank" rel="noopener noreferrer"><img
            src="./assets/svg/framework.svg"></a>
        <a href="https://edgeone.ai" target="_blank"
          rel="noopener noreferrer"><img src="./assets/svg/cdn.svg"></a>
        <a href="https://vuejs.org" target="_blank" rel="noopener noreferrer"><img src="./assets/svg/web.svg"></a>
        <a href="https://www.vvhan.com" target="_blank"><img src="./assets/svg/surppot.svg"></a>
        <a href="https://github.com/uxiaohan/HanAnalytics" target="_blank"><img src="./assets/svg/codefrom.svg"></a>
      </p>
    </footer>
  </section>
  <!-- 右下角浮动切换器 -->
  <div class="theme-switcher-float">
    <!-- 简繁体切换按钮 -->
    <button 
      :class="['theme-btn', { active: langMode !== 'n' }]"
      @click="toggleLanguage"
      :title="langMode === 't' ? '繁體中文' : (langMode === 's' ? '简体中文' : '简繁切换')"
    >
      <Languages :size="20" />
    </button>
    
    <!-- 主题切换按钮 -->
    <button 
      v-for="mode in themeModes" 
      :key="mode.value"
      :class="['theme-btn', { active: themeMode === mode.value }]"
      @click="setThemeMode(mode.value)"
      :title="mode.label"
    >
      <component :is="mode.icon" :size="20" />
    </button>
  </div>
  
  <div class="z-[999999999]">
    <Toaster />
  </div>
  <AlertDialog :open="authStatus">
    <AlertDialogContent>
      <AlertDialogHeader>
        <AlertDialogTitle>请输入登录密码</AlertDialogTitle>
        <AlertDialogDescription>
        </AlertDialogDescription>
      </AlertDialogHeader>
      <Input type="text" placeholder="Password" v-model="loginPassword" />
      <AlertDialogFooter>
        <Button :disabled="loginStatus" @click="loginFn">
          <Loader2 v-show="loginStatus" class="w-4 h-4 mr-2 animate-spin" />登录
        </Button>
      </AlertDialogFooter>
    </AlertDialogContent>
  </AlertDialog>
</template>


<script setup lang="ts">
import { ref, markRaw, onMounted, computed, watch } from 'vue'
import * as echarts from "echarts";
import { Button } from '@/components/ui/button'
import { Loader2, Sun, Moon, Clock, Languages } from 'lucide-vue-next'
import { Skeleton } from '@/components/ui/skeleton'
import { ScrollArea } from '@/components/ui/scroll-area'
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card'
import { Select, SelectContent, SelectGroup, SelectItem, SelectLabel, SelectTrigger, SelectValue, } from '@/components/ui/select'
import { Alert, AlertDescription } from '@/components/ui/alert'
import vh from 'vh-plugin'
import { Toaster } from '@/components/ui/toast'
import { useToast } from '@/components/ui/toast/use-toast'
const { toast } = useToast();
// 弹窗
import { AlertDialog, AlertDialogContent, AlertDialogDescription, AlertDialogFooter, AlertDialogHeader, AlertDialogTitle, } from '@/components/ui/alert-dialog'
import { Input } from '@/components/ui/input'

// 导入简繁转换脚本
import '@/assets/js/language.js'

// 主题模式
type ThemeMode = 'light' | 'dark' | 'auto';
const themeMode = ref<ThemeMode>((localStorage.getItem('themeMode') as ThemeMode) || 'auto');
const themeModes = [
  { value: 'light' as ThemeMode, label: '日间模式', icon: Sun },
  { value: 'dark' as ThemeMode, label: '夜间模式', icon: Moon },
  { value: 'auto' as ThemeMode, label: '自动模式', icon: Clock }
];

// 计算当前应用的主题
const currentTheme = computed(() => {
  if (themeMode.value === 'auto') {
    const hour = new Date().getHours();
    return (hour >= 6 && hour < 18) ? 'light' : 'dark';
  }
  return themeMode.value;
});

// 设置主题模式
const setThemeMode = (mode: ThemeMode) => {
  console.log('切换主题到:', mode); // 调试日志
  themeMode.value = mode;
  localStorage.setItem('themeMode', mode);
  
  // 清除旧的定时器
  if (autoThemeTimer) {
    clearInterval(autoThemeTimer);
    autoThemeTimer = null;
  }
  
  // 如果是自动模式，设置定时器
  if (mode === 'auto') {
    autoThemeTimer = setInterval(() => {
      updateEchartsTheme();
    }, 60000);
  }
  
  // 立即更新图表
  setTimeout(() => {
    updateEchartsTheme();
  }, 100);
};

// 自动模式定时器
let autoThemeTimer: number | null = null;

// 更新Echarts主题
const updateEchartsTheme = () => {
  if (canvasMain.value && tempResData.value.echarts) {
    const isDark = currentTheme.value === 'dark';
    const option = canvasMain.value.getOption();
    if (option && option.xAxis && option.xAxis[0]) {
      option.xAxis[0].axisLabel = { color: isDark ? '#9ca3af' : '#6b7280' };
      option.xAxis[0].axisLine = { 
        lineStyle: { 
          color: isDark ? 'rgba(255,255,255,0.2)' : 'rgba(0,0,0,0.1)', 
          width: 2, 
          type: 'dashed' 
        } 
      };
      option.yAxis[0].axisLabel = { color: isDark ? '#9ca3af' : '#6b7280' };
      
      // 更新线条和区域颜色
      const series = option.series[0];
      series.lineStyle = {
        width: 2,
        color: {
          colorStops: [{ offset: 1, color: isDark ? '#60a5fa' : '#6F94F1' }],
          x: 0, y: 0, x2: 1, y2: 0,
          type: 'linear',
          global: false
        }
      };
      series.areaStyle = {
        opacity: 1,
        color: {
          colorStops: [
            { offset: 0, color: isDark ? 'rgba(96, 165, 250, 0.3)' : '#DAE4FF' },
            { offset: 1, color: isDark ? 'rgba(96, 165, 250, 0.05)' : '#ffffff' }
          ],
          x: 0, y: 0, x2: 0, y2: 1,
          type: 'linear',
          global: false
        }
      };
      series.emphasis.areaStyle = {
        color: {
          colorStops: [
            { offset: 0, color: isDark ? 'rgba(96, 165, 250, 0.4)' : '#DAE4FF' },
            { offset: 1, color: isDark ? 'rgba(96, 165, 250, 0.1)' : '#ffffff' }
          ],
          x: 0, y: 0, x2: 0, y2: 1,
          type: 'linear',
          global: false
        }
      };
      
      canvasMain.value.setOption(option);
    }
  }
};

// 监听主题变化
watch(currentTheme, () => {
  updateEchartsTheme();
});

// 简繁体切换
type LangMode = 's' | 't' | 'n'; // s=简体, t=繁体, n=正常
const langMode = ref<LangMode>('n');

// 初始化语言模式
const initLanguage = () => {
  const savedLang = localStorage.getItem('zh_choose') as LangMode;
  if (savedLang) {
    langMode.value = savedLang;
  }
};

// 切换简繁体
const toggleLanguage = () => {
  // 切换顺序：正常(n) -> 繁体(t) -> 简体(s) -> 正常(n)
  if (langMode.value === 'n') {
    langMode.value = 't';
  } else if (langMode.value === 't') {
    langMode.value = 's';
  } else {
    langMode.value = 'n';
  }
  
  // 保存到localStorage
  localStorage.setItem('zh_choose', langMode.value);
  
  // 调用language.js的转换函数
  if (typeof (window as any).zh_tran === 'function') {
    (window as any).zh_tran(langMode.value);
  }
  
  console.log('%c🌏 语言切换:', 'color: #10b981; font-weight: bold;', 
    langMode.value === 't' ? '繁體中文' : (langMode.value === 's' ? '简体中文' : '正常显示'));
};

// 日期信息
const dateInfo = ref<string>('')

// 获取日期信息
const fetchDateInfo = async () => {
  try {
    console.log('%c📅 正在获取日期信息...', 'color: #3b82f6; font-weight: bold;');
    console.log('%c🔗 接口来源: https://free.xwteam.cn', 'color: #10b981; font-size: 12px;');
    
    // 使用简单请求，不设置自定义头部，避免触发CORS预检
    const response = await fetch('https://free.xwteam.cn/api/time/almanac');
    console.log('%c✅ API响应成功', `color: #10b981; font-weight: bold;`, `状态码: ${response.status}`);
    
    const data = await response.json();
    console.log('%c📦 API返回数据:', 'color: #8b5cf6; font-weight: bold;', data);
    
    if (data && data.code === 200 && data.data) {
      const apiData = data.data;
      
      // 公历：2025年12月03日 星期三
      const gongli = apiData['公历'];
      
      // 农历日期：农历 十月 十四 -> 保留"农历"，去掉多余空格
      const nongli = apiData['农历']['日期'].trim().replace(/\s+/g, '');
      
      // 天干地支：乙巳年 (蛇年) 丁亥月 丙午日 -> 保留完整内容，只去掉多余空格
      const tgdz = '[' + apiData['农历']['天干地支'].replace(/\s+/g, '') + ']';
      
      // 节日
      const jieri = apiData['节日'];
      
      console.log('%c🎯 解析结果:', 'color: #f59e0b; font-weight: bold;', { 公历: gongli, 农历: nongli, 天干地支: tgdz, 节日: jieri });
      
      if (jieri) {
        dateInfo.value = `今天是${gongli}[${jieri}]<font color="#FF0000">${nongli}${tgdz}</font>`;
      } else {
        dateInfo.value = `今天是${gongli}<font color="#FF0000">${nongli}${tgdz}</font>`;
      }
      console.log('%c✨ 最终显示:', 'color: #ec4899; font-weight: bold;', dateInfo.value);
      console.log('%c💡 喜欢这个API？访问 https://free.xwteam.cn 查看更多免费接口！', 'color: #3b82f6; font-size: 14px; font-weight: bold; background: #eff6ff; padding: 8px; border-radius: 4px;');
    } else {
      console.warn('API返回数据格式不正确', data);
      dateInfo.value = '您当前正在访问网站「Analysis」';
    }
  } catch (error) {
    console.error('获取日期信息失败:', error);
    dateInfo.value = '您当前正在访问网站「Analysis」';
  }
};

// 登录
const authStatus = ref<boolean>(false)
const session = ref<string>(localStorage.getItem('session') || '')
const loginStatus = ref<boolean>(false)
const loginPassword = ref<string>('')
const loginFn = async () => {
  if (!loginPassword.value) return toast({ description: '请输入密码', variant: 'destructive' });
  loginStatus.value = true;
  const res = await fetch('/api', { method: 'POST', headers: { 'Content-Type': 'application/json', }, body: JSON.stringify({ type: 'Login', session: loginPassword.value }) })
  await new Promise(resolve => setTimeout(resolve, 666))
  loginStatus.value = false;
  const data = await res.json()
  if (!data.success) return toast({ description: data.message, variant: 'destructive' });
  localStorage.setItem('session', loginPassword.value)
  session.value = loginPassword.value
  authStatus.value = false;
  // 站点列表
  getSiteList()
}

// 站点列表
const siteList = ref<Array<string>>([])
const siteValue = ref<string>('')
const timeList = [{ name: 'Today', value: 'today' }, { name: 'Yesterday', value: '1d' }, { name: 'Last 7 days', value: '7d' }, { name: 'Last 30 days', value: '30d' }, { name: 'Last 60 days', value: '60d' }, { name: 'Last 90 days', value: '90d' }]
const timeValue = ref<string>('today')
const getSiteList = async () => {
  vh.showLoading()
  try {
    const res = await fetch('/api', { method: 'POST', headers: { 'Content-Type': 'application/json', }, body: JSON.stringify({ type: 'list', session: session.value }) })
    const data = await res.json()
    if (data.code && data.code === 401) {
      localStorage.clear()
      authStatus.value = true
    }
    if (!data.success) return toast({ description: data.message, variant: 'destructive' });
    siteList.value = data.data;
    siteValue.value = siteList.value[0]
    if (siteValue.value) getDatas()
  } catch (error) {
    console.log(error);
  } finally {
    vh.hideLoading()
  }
}

// 获取数据
const resData = ref<any>({ visit: {} })
const tempResData = ref<any>({ visit: {} })
const getDatasStatus = ref<boolean>(false)
const getDatas = async () => {
  // 清空数据
  resData.value = { visit: {} }
  tempResData.value = { visit: {} }
  // 获取数据
  const pmsARR = ['visit', 'path', 'referrer', 'os', 'soft', 'area', 'echarts'];
  getDatasStatus.value = true
  vh.showLoading()
  const promisesForEach: Array<Promise<any>> = [];
  pmsARR.forEach((i: any) => {
    const p = new Promise((r) => {
      (async () => {
        try {
          const pms = { type: i, siteID: siteValue.value, time: timeValue.value, session: session.value }
          const res = await fetch('/api', { method: 'POST', headers: { 'Content-Type': 'application/json', }, body: JSON.stringify(pms) })
          const data = await res.json()
          if (data.code && data.code === 401) {
            localStorage.clear()
            authStatus.value = true;
          }
          if (!data.success) return toast({ description: data.message, variant: 'destructive' });
          tempResData.value[i] = i == 'echarts' ? renderEcharts(data.data.map((i: any) => `${i.name}${['today', '1d'].includes(timeValue.value) ? '点' : '日'}`), data.data.map((i: any) => `${i.value}`)) : data.data
        } catch (error) {
          console.log(error);
        } finally {
          // Promise执行完毕触发
          r(true);
        }
      })();
    });
    promisesForEach.push(p);
  })
  await Promise.all(promisesForEach);
  getDatasStatus.value = false;
  vh.hideLoading()
  // 渲染数据
  resData.value = { ...tempResData.value }
}

// 获取ICON
const getIconUrl = (url: string) => {
  if (!url) return 'https://icons.xwteam.com/ip3/none.ico'
  const _url = new URL(url)
  return `https://icons.xwteam.com/ip3/${_url.hostname}.ico`
}

// 获取Area ICON
const getIcon = (code: string) => `${location.origin}/icon/${code}.png`

// 渲染图表
const echartsDOM = ref<HTMLCanvasElement>();
const canvasMain = ref<any>();
const renderEcharts = async (dateList: Array<any>, valueList: Array<any>) => {
  const isDark = currentTheme.value === 'dark';
  const option = {
    grid: { left: "0", right: "0", bottom: "0", top: "10", containLabel: true },
    xAxis: {
      type: "category",
      data: dateList,
      axisLabel: { color: isDark ? '#9ca3af' : '#6b7280' },
      axisLine: { lineStyle: { color: isDark ? 'rgba(255,255,255,0.2)' : 'rgba(0,0,0,0.1)', width: 2, type: 'dashed' } }
    },
    yAxis: { type: "value", axisLabel: { color: isDark ? '#9ca3af' : '#6b7280' } },
    tooltip: { 
      trigger: "axis",
      backgroundColor: isDark ? '#1f2937' : '#ffffff',
      borderColor: isDark ? '#374151' : '#e5e7eb',
      textStyle: { color: isDark ? '#f3f4f6' : '#111827' }
    },
    series: [
      {
        data: valueList,
        type: "line",
        smooth: true,
        emphasis: {
          focus: "series",
          itemStyle: { borderWidth: 2 },
          areaStyle: {
            color: {
              colorStops: [
                { offset: 0, color: isDark ? 'rgba(96, 165, 250, 0.4)' : '#DAE4FF' },
                { offset: 1, color: isDark ? 'rgba(96, 165, 250, 0.1)' : '#ffffff' }
              ],
              x: 0,
              y: 0,
              x2: 0,
              y2: 1,
              type: "linear",
              global: false
            }
          }
        },
        lineStyle: {
          width: 2,
          color: {
            colorStops: [{ offset: 1, color: isDark ? '#60a5fa' : '#6F94F1' }],
            x: 0,
            y: 0,
            x2: 1,
            y2: 0,
            type: "linear",
            global: false
          }
        },
        showSymbol: false,
        areaStyle: {
          opacity: 1,
          color: {
            colorStops: [
              { offset: 0, color: isDark ? 'rgba(96, 165, 250, 0.3)' : '#DAE4FF' },
              { offset: 1, color: isDark ? 'rgba(96, 165, 250, 0.05)' : '#ffffff' }
            ],
            x: 0,
            y: 0,
            x2: 0,
            y2: 1,
            type: "linear",
            global: false
          }
        }
      }
    ]
  };
  canvasMain.value.setOption(option);
};

onMounted(() => {
  //   图表
  canvasMain.value = markRaw(echarts.init(echartsDOM.value, null, { renderer: "svg", useDirtyRect: true }));
  window.addEventListener("resize", canvasMain.value.resize);
  // 站点列表
  getSiteList();
  // 获取日期信息
  fetchDateInfo();
  // 初始化语言模式
  initLanguage();
  
  // 初始化主题
  if (themeMode.value === 'auto') {
    autoThemeTimer = setInterval(() => {
      updateEchartsTheme();
    }, 60000);
  }
});
</script>
<style>
.fixed.inset-0.z-50,
.fixed.grid.w-full.max-w-lg.shadow-lg.duration-200 {
  z-index: 99999999;
}

.han_analytics.dark {
  background: #09090b;
  color: #fafafa;
}

.han_analytics.dark > header {
  background: rgba(24, 24, 27, 0.8);
  border-bottom-color: #27272a;
}

.han_analytics.dark .git-link {
  color: #60a5fa;
}

/* 深色模式下的百分比 - 关键修复 */
.han_analytics.dark .pages-list p.page-item > em {
  color: #fafafa !important;
}

.han_analytics.dark .pages-list p.page-item:hover {
  background: #27272a;
}

.han_analytics.dark .pages-list p.page-item > em > i {
  background: rgba(96, 165, 250, 0.2);
}

/* 右下角浮动主题切换器 */
.theme-switcher-float {
  position: fixed;
  right: 24px;
  bottom: 24px;
  display: flex;
  flex-direction: column;
  gap: 12px;
  z-index: 99999999 !important;
}

.theme-switcher-float .theme-btn {
  width: 48px;
  height: 48px;
  border-radius: 50%;
  border: 1px solid #e4e4e7;
  background: #ffffff;
  color: #71717a;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.2s ease;
}

.theme-switcher-float .theme-btn:hover {
  background: #f4f4f5;
  color: #3b82f6;
  border-color: #d4d4d8;
  transform: translateY(-2px);
}

.theme-switcher-float .theme-btn.active {
  background: #3b82f6;
  color: #ffffff;
  border-color: #3b82f6;
}

/* 深色模式下的浮动按钮 */
.dark .theme-switcher-float .theme-btn {
  background: #27272a;
  border-color: #3f3f46;
  color: #a1a1aa;
}

.dark .theme-switcher-float .theme-btn:hover {
  background: #3f3f46;
  color: #60a5fa;
  border-color: #52525b;
}

.dark .theme-switcher-float .theme-btn.active {
  background: #3b82f6;
  color: #ffffff;
  border-color: #3b82f6;
}
</style>

<style scoped>
@import '@/assets/index.less';
</style>
