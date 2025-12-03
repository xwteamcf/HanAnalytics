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
type LangMode = 's' | 't' | 'n';
const langMode = ref<LangMode>('n');

// 完整的简繁转换字符对照表
const zh_s = '皑蔼碍爱翱袄奥坝罢摆败颁办绊帮绑镑谤剥饱宝报鲍辈贝钡狈备惫绷笔毕毙闭边编贬变辩辫鳖瘪濒滨宾摈饼拨钵铂驳卜补参蚕残惭惨灿苍舱仓沧厕侧册测层诧搀掺蝉馋谗缠铲产阐颤场尝长偿肠厂畅钞车彻尘陈衬撑称惩诚骋痴迟驰耻齿炽冲虫宠畴踌筹绸丑橱厨锄雏础储触处传疮闯创锤纯绰辞词赐聪葱囱从丛凑窜错达带贷担单郸掸胆惮诞弹当挡党荡档捣岛祷导盗灯邓敌涤递缔点垫电淀钓调迭谍叠钉顶锭订东动栋冻斗犊独读赌镀锻断缎兑队对吨顿钝夺鹅额讹恶饿儿尔饵贰发罚阀珐矾钒烦范贩饭访纺飞废费纷坟奋愤粪丰枫锋风疯冯缝讽凤肤辐抚辅赋复负讣妇缚该钙盖干赶秆赣冈刚钢纲岗皋镐搁鸽阁铬个给龚宫巩贡钩沟构购够蛊顾剐关观馆惯贯广规硅归龟闺轨诡柜贵刽辊滚锅国过骇韩汉阂鹤贺横轰鸿红后壶护沪户哗华画划话怀坏欢环还缓换唤痪焕涣黄谎挥辉毁贿秽会烩汇讳诲绘荤浑伙获货祸击机积饥讥鸡绩缉极辑级挤几蓟剂济计记际继纪夹荚颊贾钾价驾歼监坚笺间艰缄茧检碱硷拣捡简俭减荐槛鉴践贱见键舰剑饯渐溅涧浆蒋桨奖讲酱胶浇骄娇搅铰矫侥脚饺缴绞轿较秸阶节茎惊经颈静镜径痉竞净纠厩旧驹举据锯惧剧鹃绢杰洁结诫届紧锦仅谨进晋烬尽劲荆觉决诀绝钧军骏开凯颗壳课垦恳抠库裤夸块侩宽矿旷况亏岿窥馈溃扩阔蜡腊莱来赖蓝栏拦篮阑兰澜谰揽览懒缆烂滥捞劳涝乐镭垒类泪篱离里鲤礼丽厉励砾历沥隶俩联莲连镰怜涟帘敛脸链恋炼练粮凉两辆谅疗辽镣猎临邻鳞凛赁龄铃凌灵岭领馏刘龙聋咙笼垄拢陇楼娄搂篓芦卢颅庐炉掳卤虏鲁赂禄录陆驴吕铝侣屡缕虑滤绿峦挛孪滦乱抡轮伦仑沦纶论萝罗逻锣箩骡骆络妈玛码蚂马骂吗买麦卖迈脉瞒馒蛮满谩猫锚铆贸么霉没镁门闷们锰梦谜弥觅绵缅庙灭悯闽鸣铭谬谋亩钠纳难挠脑恼闹馁腻撵捻酿鸟聂啮镊镍柠狞宁拧泞钮纽脓浓农疟诺欧鸥殴呕沤盘庞国爱赔喷鹏骗飘频贫苹凭评泼颇扑铺朴谱脐齐骑岂启气弃讫牵扦钎铅迁签谦钱钳潜浅谴堑枪呛墙蔷强抢锹桥乔侨翘窍窃钦亲轻氢倾顷请庆琼穷趋区躯驱龋颧权劝却鹊让饶扰绕热韧认纫荣绒软锐闰润洒萨鳃赛伞丧骚扫涩杀纱筛晒闪陕赡缮伤赏烧绍赊摄慑设绅审婶肾渗声绳胜圣师狮湿诗尸时蚀实识驶势释饰视试寿兽枢输书赎属术树竖数帅双谁税顺说硕烁丝饲耸怂颂讼诵擞苏诉肃虽绥岁孙损笋缩琐锁獭挞抬摊贪瘫滩坛谭谈叹汤烫涛绦腾誊锑题体屉条贴铁厅听烃铜统头图涂团颓蜕脱鸵驮驼椭洼袜弯湾顽万网韦违围为潍维苇伟伪纬谓卫温闻纹稳问瓮挝蜗涡窝呜钨乌诬无芜吴坞雾务误锡牺袭习铣戏细虾辖峡侠狭厦锨鲜纤咸贤衔闲显险现献县馅羡宪线厢镶乡详响项萧销晓啸蝎协挟携胁谐写泻谢锌衅兴汹锈绣虚嘘须许绪续轩悬选癣绚学勋询寻驯训讯逊压鸦鸭哑亚讶阉烟盐严颜阎艳厌砚彦谚验鸯杨扬疡阳痒养样瑶摇尧遥窑谣药爷页业叶医铱颐遗仪彝蚁艺亿忆义诣议谊译异绎荫阴银饮樱婴鹰应缨莹萤营荧蝇颖哟拥佣痈踊咏涌优忧邮铀犹游诱舆鱼渔娱与屿语吁御狱誉预驭鸳渊辕园员圆缘远愿约跃钥岳粤悦阅云郧匀陨运蕴酝晕韵杂灾载攒暂赞赃脏凿枣灶责择则泽贼赠扎札轧铡闸诈斋债毡盏斩辗崭栈战绽张涨帐账胀赵蛰辙锗这贞针侦诊镇阵挣睁狰帧郑证织职执纸挚掷帜质钟终种肿众诌轴皱昼骤猪诸诛烛瞩嘱贮铸筑驻专砖转赚桩庄装妆壮状锥赘坠缀谆浊兹资渍踪综总纵邹诅组钻致钟么为只凶准启板里雳余链泄';
const zh_t = '皚藹礙愛翺襖奧壩罷擺敗頒辦絆幫綁鎊謗剝飽寶報鮑輩貝鋇狽備憊繃筆畢斃閉邊編貶變辯辮鼈癟瀕濱賓擯餅撥缽鉑駁蔔補參蠶殘慚慘燦蒼艙倉滄廁側冊測層詫攙摻蟬饞讒纏鏟産闡顫場嘗長償腸廠暢鈔車徹塵陳襯撐稱懲誠騁癡遲馳恥齒熾沖蟲寵疇躊籌綢醜櫥廚鋤雛礎儲觸處傳瘡闖創錘純綽辭詞賜聰蔥囪從叢湊竄錯達帶貸擔單鄲撣膽憚誕彈當擋黨蕩檔搗島禱導盜燈鄧敵滌遞締點墊電澱釣調叠諜疊釘頂錠訂東動棟凍鬥犢獨讀賭鍍鍛斷緞兌隊對噸頓鈍奪鵝額訛惡餓兒爾餌貳發罰閥琺礬釩煩範販飯訪紡飛廢費紛墳奮憤糞豐楓鋒風瘋馮縫諷鳳膚輻撫輔賦複負訃婦縛該鈣蓋幹趕稈贛岡剛鋼綱崗臯鎬擱鴿閣鉻個給龔宮鞏貢鈎溝構購夠蠱顧剮關觀館慣貫廣規矽歸龜閨軌詭櫃貴劊輥滾鍋國過駭韓漢閡鶴賀橫轟鴻紅後壺護滬戶嘩華畫劃話懷壞歡環還緩換喚瘓煥渙黃謊揮輝毀賄穢會燴彙諱誨繪葷渾夥獲貨禍擊機積饑譏雞績緝極輯級擠幾薊劑濟計記際繼紀夾莢頰賈鉀價駕殲監堅箋間艱緘繭檢堿鹼揀撿簡儉減薦檻鑒踐賤見鍵艦劍餞漸濺澗漿蔣槳獎講醬膠澆驕嬌攪鉸矯僥腳餃繳絞轎較稭階節莖驚經頸靜鏡徑痙競淨糾廄舊駒舉據鋸懼劇鵑絹傑潔結誡屆緊錦僅謹進晉燼盡勁荊覺決訣絕鈞軍駿開凱顆殼課墾懇摳庫褲誇塊儈寬礦曠況虧巋窺饋潰擴闊蠟臘萊來賴藍欄攔籃闌蘭瀾讕攬覽懶纜爛濫撈勞澇樂鐳壘類淚籬離裏鯉禮麗厲勵礫曆瀝隸倆聯蓮連鐮憐漣簾斂臉鏈戀煉練糧涼兩輛諒療遼鐐獵臨鄰鱗凜賃齡鈴淩靈嶺領餾劉龍聾嚨籠壟攏隴樓婁摟簍蘆盧顱廬爐擄鹵虜魯賂祿錄陸驢呂鋁侶屢縷慮濾綠巒攣孿灤亂掄輪倫侖淪綸論蘿羅邏鑼籮騾駱絡媽瑪碼螞馬罵嗎買麥賣邁脈瞞饅蠻滿謾貓錨鉚貿麽黴沒鎂門悶們錳夢謎彌覓綿緬廟滅憫閩鳴銘謬謀畝鈉納難撓腦惱鬧餒膩攆撚釀鳥聶齧鑷鎳檸獰甯擰濘鈕紐膿濃農瘧諾歐鷗毆嘔漚盤龐國愛賠噴鵬騙飄頻貧蘋憑評潑頗撲鋪樸譜臍齊騎豈啓氣棄訖牽扡釺鉛遷簽謙錢鉗潛淺譴塹槍嗆牆薔強搶鍬橋喬僑翹竅竊欽親輕氫傾頃請慶瓊窮趨區軀驅齲顴權勸卻鵲讓饒擾繞熱韌認紉榮絨軟銳閏潤灑薩鰓賽傘喪騷掃澀殺紗篩曬閃陝贍繕傷賞燒紹賒攝懾設紳審嬸腎滲聲繩勝聖師獅濕詩屍時蝕實識駛勢釋飾視試壽獸樞輸書贖屬術樹豎數帥雙誰稅順說碩爍絲飼聳慫頌訟誦擻蘇訴肅雖綏歲孫損筍縮瑣鎖獺撻擡攤貪癱灘壇譚談歎湯燙濤縧騰謄銻題體屜條貼鐵廳聽烴銅統頭圖塗團頹蛻脫鴕馱駝橢窪襪彎灣頑萬網韋違圍爲濰維葦偉僞緯謂衛溫聞紋穩問甕撾蝸渦窩嗚鎢烏誣無蕪吳塢霧務誤錫犧襲習銑戲細蝦轄峽俠狹廈鍁鮮纖鹹賢銜閑顯險現獻縣餡羨憲線廂鑲鄉詳響項蕭銷曉嘯蠍協挾攜脅諧寫瀉謝鋅釁興洶鏽繡虛噓須許緒續軒懸選癬絢學勳詢尋馴訓訊遜壓鴉鴨啞亞訝閹煙鹽嚴顔閻豔厭硯彥諺驗鴦楊揚瘍陽癢養樣瑤搖堯遙窯謠藥爺頁業葉醫銥頤遺儀彜蟻藝億憶義詣議誼譯異繹蔭陰銀飲櫻嬰鷹應纓瑩螢營熒蠅穎喲擁傭癰踴詠湧優憂郵鈾猶遊誘輿魚漁娛與嶼語籲禦獄譽預馭鴛淵轅園員圓緣遠願約躍鑰嶽粵悅閱雲鄖勻隕運蘊醞暈韻雜災載攢暫贊贓髒鑿棗竈責擇則澤賊贈紮劄軋鍘閘詐齋債氈盞斬輾嶄棧戰綻張漲帳賬脹趙蟄轍鍺這貞針偵診鎮陣掙睜猙幀鄭證織職執紙摯擲幟質鍾終種腫衆謅軸皺晝驟豬諸誅燭矚囑貯鑄築駐專磚轉賺樁莊裝妝壯狀錐贅墜綴諄濁茲資漬蹤綜總縱鄒詛組鑽緻鐘麼為隻兇準啟闆裡靂餘鍊洩';

// 字符串转换函数
const convertText = (text: string, mode: LangMode): string => {
  if (mode === 'n') return text;
  const sourceChars = mode === 't' ? zh_s : zh_t;
  const targetChars = mode === 't' ? zh_t : zh_s;
  let result = '';
  for (let i = 0; i < text.length; i++) {
    const char = text.charAt(i);
    const index = sourceChars.indexOf(char);
    result += index >= 0 ? targetChars.charAt(index) : char;
  }
  return result;
};

// 转换DOM节点
const convertNode = (node: Node, mode: LangMode) => {
  if (node.nodeType === Node.TEXT_NODE) {
    if (node.textContent) {
      node.textContent = convertText(node.textContent, mode);
    }
  } else if (node.nodeType === Node.ELEMENT_NODE) {
    const element = node as HTMLElement;
    if (element.title) element.title = convertText(element.title, mode);
    if (element.getAttribute('alt')) {
      element.setAttribute('alt', convertText(element.getAttribute('alt')!, mode));
    }
    if (['SCRIPT', 'STYLE', 'TEXTAREA', 'INPUT'].indexOf(element.tagName) === -1) {
      node.childNodes.forEach(child => convertNode(child, mode));
    }
  }
};

// 初始化语言模式
const initLanguage = () => {
  const savedLang = localStorage.getItem('zh_choose') as LangMode;
  if (savedLang && savedLang !== 'n') {
    langMode.value = savedLang;
    setTimeout(() => {
      convertNode(document.body, savedLang);
    }, 100);
  }
};

// 切换简繁体
const toggleLanguage = () => {
  if (langMode.value === 'n') {
    langMode.value = 't';
  } else if (langMode.value === 't') {
    langMode.value = 's';
  } else {
    langMode.value = 'n';
  }
  localStorage.setItem('zh_choose', langMode.value);
  if (langMode.value === 'n') {
    window.location.reload();
  } else {
    convertNode(document.body, langMode.value);
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
