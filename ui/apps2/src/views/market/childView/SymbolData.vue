<template>
  <div class="symbolData">
    <div class="myPools">
      <!-- <div class="title"><span class="act">{{ $t('mine.myMine') }}</span></div> -->
      <div class="title"><span class="act">{{ $t('mine.symbolPool', {symbol: `${thisMarket.symbol0}-${thisMarket.symbol1}`}) }}</span></div>
      <div class="data">
        <div class="allClaim flexb">
          <div>
            <div class="subTitle flexa">
              <span>{{ $t('mine.waitClaim') }}</span>
              <img class="tipIcon ml10" @click="showReWardTip = true" src="@/assets/img/dex/tips_icon_btn.svg" alt="">
            </div>
            <div class="claimNum">{{ accMineData.showReward || '0.00000000' }} DFS</div>
          </div>
          <div class="allClaimBtn" v-if="Number(accMineData.liq)" v-loading="claimLoading"
            @click="handleClaim(thisMarket)">{{ $t('bonus.claim') }}</div>
          <div class="allClaimBtn" v-if="!Number(accMineData.liq) && getAccData"
            @click="handleJoin(thisMarket)">{{ $t('mine.join') }}</div>
        </div>
        <div :class="`mylist ${handleGetClass(thisMarket.mid)}`" @click="handleJoin(thisMarket)">
          <div class="flexb">
            <span class="flexa" v-if="handleGetClass(thisMarket.mid) === '' && Number(buff)">
              <img class="buffImg" src="@/assets/img/poolspage/buff2.svg">
              <span class="addition">{{ $t('mine.buff') }}：{{ buff }}%</span>
            </span>
          </div>
          <div class="symbol flexb" :class="{'noTop': !(handleGetClass(thisMarket.mid) === '' && Number(buff))}">
            <div class="coinInfo flex">
              <div class="coinImg"><img width="100%" :src="thisMarket.sym0Data.imgUrl" :onerror="errorCoinImg"></div>
              <div>
                <div class="coin">{{ thisMarket.symbol0 }}</div>
                <div class="contract tip">{{ thisMarket.contract0 }}</div>
              </div>
            </div>
            <div class="add">+</div>
            <div class="coinInfo flex">
              <div class="coinImg"><img width="100%" :src="thisMarket.sym1Data.imgUrl" :onerror="errorCoinImg"></div>
              <div>
                <div class="coin">{{ thisMarket.symbol1 }}</div>
                <div class="contract tip">{{ thisMarket.contract1 }}</div>
              </div>
            </div>
          </div>
        </div>
        <div :class="`tipDiv marketReward ${handleGetClass(thisMarket.mid)}`"
          v-loading="nowMarketLoading"
          v-if="Number(nowMarket.getNum1) || marketData.length">
          <div class="flexa">
            <span>{{ $t('mine.accPools') }}: </span>
            <span>{{ nowMarket.getNum1 || `0.0000` }} {{thisMarket.symbol0}} / {{ nowMarket.getNum2 || `0.0000`}} {{thisMarket.symbol1}}</span>
          </div>
          <div class="flex">
            <span>{{ $t('market.capital') }}: </span>
            <span v-if="!marketData.length" class="tip maxW">
              <span>{{ $t('market.anthorOne') }} </span>
              <span class="green" @click="handleJoin(thisMarket)">{{ $t('market.rightNow') }}</span>
            </span>
            <span v-else>{{ `${marketData[0]} ${thisMarket.symbol0}` }} / {{ `${marketData[1]} ${thisMarket.symbol1}` }}</span>
          </div>
          <div class="flexa">
            <span>{{ $t('market.marketReward') }}: </span>
            <span :class="{'green': sym0AndSy1 ? parseFloat(marketRewardSym0) > 0 : parseFloat(marketRewardSym1) > 0,
                          'red': sym0AndSy1 ? parseFloat(marketRewardSym0) < 0 : parseFloat(marketRewardSym1) < 0}">
              {{ sym0AndSy1 ? marketRewardSym0 : marketRewardSym1 }}
            </span>
            <span :class="{'green':  sym0AndSy1 ? parseFloat(marketRewardSym1) > 0 : parseFloat(marketRewardSym0) > 0,
                          'red':  sym0AndSy1 ? parseFloat(marketRewardSym1) < 0 : parseFloat(marketRewardSym0) < 0}">
              ({{ sym0AndSy1 ? marketRewardSym1 : marketRewardSym0 }})
            </span>
            <img class="qusTip" src="@/assets/img/dex/tips_icon_btn.svg" @click="showMarketTip = !showMarketTip">
          </div>
          <div class="flexa">
            <span>{{ $t('market.marketTime') }}: </span>
            <span>{{ $t('market.timer', {
                days: marketTime.days,
                hours: marketTime.hours,
                mins: marketTime.minutes,
                secs: marketTime.seconds
              }) }}</span>
            <span class="tip">（{{ $t('market.pl') }}: 
              <span :class="{'green': Number(percent) > 0, 'red': Number(percent < 0)}">
                {{ percent }}%
              </span>）
            </span>
          </div>
        </div>
        <div :class="`tipDiv ${handleGetClass(thisMarket.mid)}`">
          <div class="">
            <span>{{ $t('dex.pools') }}: </span>
            <span>{{ thisMarket.reserve0 || '—' }} / {{ thisMarket.reserve1 || '—' }}</span>
          </div>
          <div class="tip rewardPerDay allApr">
            <span>预估24H年化: <b>{{ countApy }}%</b></span>
            <span class="green" @click.stop="showApyDetail = true">详情></span>
          </div>
          <div class="rewardPerDay tip">
            <span>{{ $t('mine.poolsMine2', {perDayReward: dayRewardNum}) }}</span>
          </div>
          <!-- <div class="rewardPerDay tip"><span>{{ $t('mine.mineApr') }}: {{ apr }}%</span></div>
          <div class="rewardPerDay tip" v-if="!isActual && Number(feesApr)"><span>{{ $t('mine.marketFeesApr') }}: {{ feesApr }} %</span></div>
          <div class="rewardPerDay tip" v-if="isActual && Number(feesApr)"><span>{{ $t('mine.marketApr24H') }}: {{ feesApr }} %</span></div>
          <div class="rewardPerDay tip" v-if="Number(yfcApy)">YFC钓鱼年化: {{ yfcApy }}%</div> -->
        </div>
      </div>
    </div>

    <div class="poolsLists">
      <div class="title flexb">
        <span class="act">{{ $t('mine.minersList') }}</span>
        <span class="totalMiners">{{ $t('mine.totalMiners') }}：{{ allMinersList.length }}</span>
      </div>
      <div class="noData" v-loading="!getMinersList" v-if="!minersArr.length">{{ $t('public.noData') }}</div>
      <template v-for="(item, index) in minersArr">
        <div class="list" :class="{'page1': page === 1}" :key="index">
          <div class="flexb mb10">
            <span>{{ item.miner }}</span>
            <span>{{ $t('mine.earnings') }}：{{ item.showReward || '0.00000000' }} DFS</span>
          </div>
          <div class="flexb">
            <span>{{ $t('dex.pools') }}</span>
            <span>{{ item.liq_bal0 }} / {{ item.liq_bal1 }}</span>
          </div>
          <label class="rankImg" v-if="page === 1 && index < 3"><img :src="`https://apps.defis.network/static/rank/rank${index + 1}.png`" alt=""></label>
        </div>
      </template>
      <el-pagination
        v-if="allMinersList.length"
        class="pagination"
        layout="prev, pager, next"
        @current-change="handleCurrentChange"
        :current-page.sync="page"
        :page-size="pageSize"
        :total="allMinersList.length">
      </el-pagination>
    </div>

    <el-dialog
      class="myDialog"
      :visible.sync="showReWardTip">
      <MinReward :minReward="minReward"/>
    </el-dialog>
    <el-dialog
      class="myDialog"
      :visible.sync="showMarketTip">
      <MarketTip v-if="showMarketTip"/>
    </el-dialog>
    <el-dialog
      class="myDialog apy"
      :visible.sync="showApyDetail">
      <MarketApy :countApy="countApy" :feesApr="feesApr" :isActual="isActual"
                 :apr="apr" :lpApy="lpApy" :dmdApy="dmdApy"/>
    </el-dialog>
  </div>
</template>

<script>
import axios from "axios";
import { mapState } from 'vuex';
import { EosModel } from '@/utils/eos';
import { toFixed, accSub, accAdd, accMul, accDiv, dealReward, getMarketTime,
         dealMinerData, perDayReward, getPoolApr, getClass, getYfcReward, getDmdMinerHourRoi } from '@/utils/public';
import { sellToken } from '@/utils/logic';
import MinReward from '../popup/MinReward'
import MarketTip from '../popup/MarketTip';
import MarketApy from '../popup/MarketApy'

export default {
  components: {
    MinReward,
    MarketTip,
    MarketApy
  },
  data() {
    return {
      errorCoinImg: 'this.src="https://ndi.340wan.com/eos/eosio.token-eos.png"',
      getMinersList: false,
      showReWardTip: false,
      claimLoading: false,
      getAccData: false,
      thisMarket: {
        symbol0: 'EOS',
        symbol1: 'USDD',
        contract0: 'eosio.token',
        contract1: 'bankofusddv1',
        sym0Data: {
          imgUrl: "https://apps.defis.network/static/coin/eosio.token-eos.svg"
        },
        sym1Data: {
          imgUrl: "https://apps.defis.network/static/coin/bankofusddv1-usdd.svg"
        },
      }, // 当前矿池数据
      accMineData: {}, // 用户记录
      allMinersList: [], // 所有挖矿者列表
      page: 1,
      pageSize: 20,
      minersArr: [], // 所有挖矿者列表 - 单页面
      timerArr: [], // 所有挖矿者收益定时器
      secTimer: null, // 秒级定时器
      accTimer: null, // 用户自己的收益定时器
      accSecTimer: null, // 用户自己的秒级定时器
      token: '0',
      marketData: [], // 做市本金
      direction: true,
      nowMarket: {}, // 实时做市资金
      nowMarketLoading: true,
      timer: null,
      marketTime: {
        days: 0,
        hours: '00',
        minutes: '00',
        seconds: '00'
      },
      showMarketTip: false,
      showApyDetail: false,
      lpApy: {},
    }
  },
  props: {
    marketLists: {
      type: Array,
      default: function lists() {
        return []
      }
    }
  },
  watch: {
    marketLists: {
      handler: function ml(newVal) {
        if (!newVal.length) {
          return
        }
        this.thisMarket = newVal.find(v => v.mid === Number(this.$route.params.mid))
        this.handleGetNowMarket()
      },
      immediate: true
    },
    scatter: {
      handler: function listen(newVal) {
        if (newVal.identity) {
          this.handleGetMinersLists('user')
          this.handleGetMarketData()
          this.handleGetAccToken()
        }
      },
      deep: true,
      immediate: true,
    },
  },
  computed: {
    ...mapState({
      // 箭头函数可使代码更简练
      baseConfig: state => state.sys.baseConfig, // 基础配置 - 默认为{}
      weightList: state => state.sys.weightList, // 交易对权重列表
      aprs: state => state.sys.aprs,
      damping: state => state.sys.damping,
      scatter: state => state.app.scatter,
      dfsPrice: state => state.sys.dfsPrice,
      storeFeesApr: state => state.sys.feesApr,
      lpMid: state => state.config.lpMid,
    }),
    dmdApy() {
      const dmdPool = this.marketLists.find(v => v.mid === 326)
      let dmdRoi = getDmdMinerHourRoi(this.thisMarket, 'year', dmdPool)
      if (Number(dmdRoi)) {
        return dmdRoi;
      }
      return '0.000';
    },
    countApy() {
      let all = accAdd(parseFloat(this.apr), parseFloat(this.feesApr))
      if (this.dmdApy) {
        all = accAdd(all, parseFloat(this.dmdApy))
      }
      this.lpMid.forEach(v => {
        const apy = this.handleDealApy(v.mid, v.symbol);
        this.lpApy[`${v.symbol.toLowerCase()}Apy`] = apy;
        if (apy) {
          all = accAdd(all, Number(apy))
        }
      })
      if (isNaN(all)) {
        return '—'
      }
      return all.toFixed(2)
    },
    minReward() {
      if (!Number(this.dfsPrice)) {
        return '0.0005'
      }
      let min = accDiv(0.0001, this.dfsPrice)
      if (Number(toFixed(min, 4)) < min) {
        min = accAdd(min, 0.0001)
      }
      return toFixed(min, 4)
    },
    weight() {
      if (!this.weightList.length || !this.thisMarket.mid) {
        return '0'
      }
      const wData = this.weightList.find(v => v.mid === this.thisMarket.mid) || {}
      return wData.pool_weight;
    },
    buff() {
      if (Number(this.weight) <= 1) {
        return 0
      }
      let t = accSub(this.weight, 1);
      t = accMul(t, 100);
      return t.toFixed(0)
    },
    dayRewardNum() {
      if (Number(this.weight) <= 0) {
        return '0.0000'
      }
      return perDayReward(this.weight)
    },
    apr() {
      const apr = this.dayRewardNum * this.dfsPrice / 20000 * 365 * 100;
      return apr.toFixed(2)
    },
    feesApr() {
      const feesApr = this.storeFeesApr.find(v => v.symbol === this.thisMarket.symbol1) || {}
      const thisPoolApr = getPoolApr(this.thisMarket)
      return parseFloat(feesApr.poolsApr) > parseFloat(thisPoolApr) ? parseFloat(feesApr.poolsApr) : parseFloat(thisPoolApr)
      // return getPoolApr(this.thisMarket)
    },
    isActual() {
      const feesApr = this.storeFeesApr.find(v => v.symbol === this.thisMarket.symbol1) || {}
      const thisPoolApr = getPoolApr(this.thisMarket)
      return parseFloat(feesApr.poolsApr) > parseFloat(thisPoolApr)
    },
    marketReward() {
      if (!this.marketData.length || !Number(this.nowMarket.getNum1)) {
        return '0.0000';
      }
      const sym0 = accSub(parseFloat(this.nowMarket.getNum1), this.marketData[0]);
      const sym1 = accSub(parseFloat(this.nowMarket.getNum2), this.marketData[1]);
      const price = accDiv(parseFloat(this.nowMarket.getNum2), parseFloat(this.nowMarket.getNum1));
      if (this.direction) {
        const reward = sym1 + sym0 * price;
        return toFixed(reward, this.thisMarket.decimal1)
      }
      const reward = sym0 + sym1 / price;
      return toFixed(reward, this.thisMarket.decimal0)
    },
    marketRewardSym0() {
      return this.handleDealReward('sym0')
    },
    marketRewardSym1() {
      return this.handleDealReward('sym1')
    },
    sym0AndSy1() {
      return parseFloat(this.marketRewardSym0) > parseFloat(this.marketRewardSym1)
    },
    percent() {
      if (!this.marketData.length || !Number(this.nowMarket.getNum1)) {
        return '0.0000';
      }
      const sym0 = accSub(parseFloat(this.nowMarket.getNum1), this.marketData[0]);
      const sym1 = accSub(parseFloat(this.nowMarket.getNum2), this.marketData[1]);
      const price = accDiv(parseFloat(this.nowMarket.getNum2), parseFloat(this.nowMarket.getNum1));
      const reward = sym0 + sym1 / price;
      let mD = accMul(this.marketData[0], 2)
      let p = accDiv(reward, mD)
      p = accMul(p, 100);
      return toFixed(p, 2)
    }
  },
  mounted() {
    this.handleGetMinersLists()
  },
  beforeDestroy() {
    clearTimeout(this.timer)
    clearInterval(this.accTimer)
    clearInterval(this.secTimer)
    clearInterval(this.accSecTimer)
    this.timerArr.forEach(v => {
      clearInterval(v)
    })
  },
  methods: {
    handleDealApy(mid = 329, project) {
      let yfcReward = getYfcReward(this.thisMarket.mid, 'year', project);
      if (Number(yfcReward)) {
        const YfcPool = this.marketLists.find(vv => vv.mid === mid);
        const price = parseFloat(YfcPool.reserve0 || 0) / parseFloat(YfcPool.reserve1 || 1)
        yfcReward = yfcReward * price / 20000 * 100;
        yfcReward = yfcReward.toFixed(2);
      }
      return yfcReward || '0.00';
    },
    handleDealReward(sym) {
      if (!this.marketData.length || !Number(this.marketData[0])  || !this.nowMarket.getNum1) {
        return `0.0000 ${sym === 'sym1'? this.thisMarket.symbol1 : this.thisMarket.symbol0}`;
      }
      if (sym === 'sym1') {
        const sym1 = accSub(parseFloat(this.nowMarket.getNum2), this.marketData[1]);
        const t = sym1 > 0 ? '+' : ''
        return `${t}${toFixed(sym1, this.thisMarket.decimal1)} ${this.thisMarket.symbol1}`
      }
      const t = sym0 > 0 ? '+' : ''
      const sym0 = accSub(parseFloat(this.nowMarket.getNum1), this.marketData[0]);
      return `${t}${toFixed(sym0, this.thisMarket.decimal0)} ${this.thisMarket.symbol0}`
    },
    handleGetTime() {
      clearTimeout(this.timer)
      if (Number(this.sTime)) {
        this.marketTime = getMarketTime(this.sTime)
        this.timer = setTimeout(() => {
          this.handleGetTime()
        }, 1000);
        return
      }
      this.marketTime = {
        days: 0,
        hours: '00',
        minutes: '00',
        seconds: '00'
      }
    },
    handleGetAccToken() {
      const params = {
        code: this.baseConfig.toAccountSwap,
        scope: this.$route.params.mid,
        table: 'liquidity',
        lower_bound: ` ${this.scatter.identity.accounts[0].name}`,
        upper_bound: ` ${this.scatter.identity.accounts[0].name}`,
        json: true
      }
      EosModel.getTableRows(params, (res) => {
        const list = res.rows || [];
        !list[0] ? this.token = '0' : this.token = `${list[0].token}`;
        this.handleGetNowMarket()
      })
    },
    handleGetNowMarket() {
      try {
        const inData = {
          poolSym0: this.thisMarket.reserve0.split(' ')[0],
          poolSym1: this.thisMarket.reserve1.split(' ')[0],
          poolToken: this.thisMarket.liquidity_token,
          sellToken: this.token
        }
        const nowMarket = sellToken(inData);
        nowMarket.getNum1 = toFixed(nowMarket.getNum1, 4)
        nowMarket.getNum2 = toFixed(nowMarket.getNum2, 4)
        this.nowMarket = nowMarket;
        this.nowMarketLoading = false;
      } catch(error) {
        setTimeout(() => {
          this.handleGetNowMarket()
        }, 200);
      }
    },
    handleGetMarketData() {
      const params = {
        user: this.scatter.identity.accounts[0].name,
        mid: this.$route.params.mid,
      }
      axios.get('https://dfsinfoapi.sgxiang.com/dapi/changelogdata', {params}).then((result) => {
        const res = result.data;
        this.marketData = [];
        this.sTime = '0'
        if (!result.data.logs.length) {
          return
        }
        const newArr = []
        for (const key in res) {
          if (key !== 'logs' && key !== 'tag_log_format_block_time' && key !== 'tag_log_utc_block_time') {
            if (key !== 'EOS') {
              newArr.push(toFixed(res[key], this.thisMarket.decimal1 || 4))
            } else {
              newArr.unshift(toFixed(res[key], this.thisMarket.decimal0 || 4))
            }
          }
        }
        this.sTime = res.tag_log_utc_block_time;
        this.handleGetTime();
        this.marketData = newArr;
      })
    },
    handleGetClass(mid) {
      return getClass(mid)
    },
    handleCurrentChange() {
      this.handleGetPageArr();
    },
    handleGetMinersLists(type) {
      const params = {
        "code": "miningpool11",
        "scope": this.$route.params.mid,
        "table": "miners",
        // "lower_bound": " dfsdeveloper",
        // "upper_bound": " dfsdeveloper",
        limit: 2000,
        "json": true,
      }
      if (type === 'user') {
        params.lower_bound = ` ${this.scatter.identity.accounts[0].name}`;
        params.upper_bound = ` ${this.scatter.identity.accounts[0].name}`;
      }
      EosModel.getTableRows(params, (res) => {
        if (type === 'user') {
          this.getAccData = true;
        } else {
          this.getMinersList = true;
        }
        const rows = res.rows || []
        if (!rows.length) {
          this.accMineData = {};
          return
        }
        const newList = [];
        rows.forEach(item => {
          let v = item;
          // if (this.thisMarket.exchangeSym) {
          if (v.liq_bal1.split(' ')[1] === 'EOS' || this.thisMarket.exchangeSym) {
            const tList = {
              last_drip: v.last_drip,
              liq_bal0: v.liq_bal1,
              liq_bal1: v.liq_bal0,
              miner: v.miner
            }
            v = tList;
          }
          const minnerData = dealMinerData(v, this.thisMarket)
          // console.log(minnerData)
          if (type === 'user') {
            this.accMineData = minnerData;
            return;
          }
          // if (this.scatter.identity && this.scatter.identity.accounts[0].name === v.miner) {
          //   return
          // }
          newList.push(minnerData)
        })
        if (type === 'user') {
          this.handleRunAccReward()
          return
        }
        const newListSort = newList.sort((a, b) => {
          return b.liq - a.liq;
        })
        try {
          this.allMinersList = newListSort;
          this.handleGetPageArr();
        } catch (error) {
          console.log(error)
        }
      })
    },
    handleGetPageArr() {
      const start = (this.page - 1) * this.pageSize;
      const end = this.page * this.pageSize;
      this.minersArr = this.allMinersList.slice(start, end);
      // console.log(this.minersArr)
      this.handleRunReward()
    },
    // 秒级定时器
    handleRunReward() {
      clearInterval(this.secTimer)
      this.handleRunLogic()
      this.secTimer = setInterval(() => {
        this.handleRunLogic()
      }, 1000);
    },
    // 数据滚动效果
    handleRunLogic() {
      this.minersArr.forEach((v, index) => {
        if (this.timerArr[index]) {
          clearInterval(this.timerArr[index]);
        }
        if (!Number(v.liq)) {
          this.timerArr[index] = null;
          return
        }
        const reward = dealReward(v, this.weight)
        let showReward = v.reward || '0.00000000';
        let countReward = showReward;
        if (!v.showReward) {
          this.$set(v, 'showReward', reward)
          showReward = reward;
          countReward = reward;
        }
        this.$set(v, 'reward', reward)
        let t = accSub(reward, showReward);
        t = accDiv(t, 20)
        this.timerArr[index] = setInterval(() => {
          countReward = accAdd(countReward, t)
          if (countReward > Number(reward)) {
            showReward = toFixed(reward, 8);
            clearInterval(this.timerArr[index])
          } else {
            showReward = toFixed(countReward, 8);
          }
          this.$set(v, 'showReward', showReward);
        }, 50);
      })
    },
    handleRunAccReward() {
      clearInterval(this.accSecTimer)
      this.handleAccRun()
      this.accSecTimer = setInterval(() => {
        this.handleAccRun()
      }, 1000);
    },
    handleAccRun() {
      // console.log(this.accMineData)
      const reward = dealReward(this.accMineData, this.weight)
      let showReward = this.accMineData.reward || '0.00000000';
      let countReward = showReward;
      if (!this.accMineData.showReward) {
        this.accMineData.showReward = reward;
        showReward = reward;
        countReward = reward;
      }
      this.accMineData.reward = reward;
      let t = accSub(reward, showReward);
      t = accDiv(t, 20)
      clearInterval(this.accTimer)
      this.accTimer = setInterval(() => {
        countReward = accAdd(countReward, t)
        if (countReward > Number(reward)) {
          showReward = toFixed(reward, 8);
          clearInterval(this.accTimer)
        } else {
          showReward = toFixed(countReward, 8);
        }
        this.accMineData.showReward = showReward;
        // console.log(this.accMineData.showReward)
      }, 50);
    },
    handleJoin(item) {
      this.$router.push({
        name: 'market',
        params: {
          mid: item.mid
        }
      })
    },
    handleClaim(item) {
      if (this.claimLoading) {
        return
      }
      if (Number(this.accMineData.reward) < Number(this.minReward)) {
        this.showReWardTip = true;
        return
      }
      this.claimLoading = true;
      const formName = this.$store.state.app.scatter.identity.accounts[0].name;
      const permission = this.$store.state.app.scatter.identity.accounts[0].authority;
      const params = {
        actions: [
          {
            account: 'miningpool11',
            name: 'claim',
            authorization: [{
              actor: formName, // 转账者
              permission,
            }],
            data: {
              user: formName,
              mid: item.mid,
            }
          },
        ]
      }
      EosModel.toTransaction(params, (res) => {
        this.claimLoading = false
        if(res.code && JSON.stringify(res.code) !== '{}') {
          this.$message({
            message: res.message,
            type: 'error'
          });
          return
        }
        // this.changeReWard = toFixed(0, 8);
        // this.reward = 0;
        this.handleGetMinersLists('user')
        this.$message({
          message: this.$t('public.success'),
          type: 'success'
        });
      })
    },
  },
}
</script>

<style lang="scss" scoped>
.title{
  font-size: 32px;
  text-align: left;
  margin: 0 0 40px;
  &>span{
    margin-right: 60px;;
  }
  .act{
    color: $color-black;
    position: relative;
    &::before{
      content: '';
      position: absolute;
      width:60px;
      height:8px;
      background:rgba(2,198,152,1);
      border-radius:4px;
      bottom: -20px;
      left: 50%;
      transform: translateX(-45%);
    }
  }
}
.green{
  color: #07D79B;
}
.red{
  color: #E9574F;
}
.ml10{
  margin-left: 10px;
}
.mylist{
  margin-top: 20px;
  padding: 20px;
  border: 1px solid #e0e0e0;
  border-radius: 20px;
  &.gold {
    border: 1px solid rgb(238, 198, 4);
    box-shadow: 0 0 5px 0px rgba(238, 198, 4, .5);
  }
  &.silver {
    border: 1px solid #c0c0c0;
    box-shadow: 0 0 5px 0px rgba(#c0c0c0, .5);
  }
  &.bronze {
    border: 1px solid #8C7853;
    box-shadow: 0 0 5px 0px rgba(#8C7853, .5);
  }
  .addition{
    font-size: 24px;
    color: #E9574F;
  }
  .buffImg{
    width: 30px;
    margin-right: 8px;
  }
}
.symbolData{
  text-align: left;
  font-size: 28px;
  color: #000;
  .allClaim{
    // margin: 0 40px 40px;
    background: #07D79B;
    border-radius: 20px;
    color: #FFF;
    padding: 40px;
    .ml10{
      margin-left: 10px;
    }
    .subTitle{
      font-size: 26px;
    }
    .claimNum{
      font-size: 30px;
      font-weight: 500;
      margin-top: 8px;
    }
    .allClaimBtn{
      background: #FFF;
      border-radius: 8px;
      color: #07D79B;
      font-size: 28px;
      padding: 10px 30px;
    }
  }
  .myPools{
    margin: 20px 40px 40px;
    .addition{
      font-size: 24px;
      color: #E9574F;
    }
    .symbol{
      margin-top: 20px;
      border-radius: 20px;
      &.noTop{
        margin-top: 0px;
      }
      .coinInfo{
        text-align: left;
        .coinImg{
          width: 60px;
          height: 60px;
          margin-right: 10px;
          border-radius: 60px;
          overflow: hidden;
        }
        .coin{
          font-size: 28px;
          font-weight: 500;;
          line-height: 30px;
        }
        .contract{
          line-height: 30px;
        }
        .ableGet{
          color: #2F3F52;
        }
      }
      .add{
        width: 40px;
      }
    }
    .myMarket{
      margin-top: 10px;
    }
    .rewardPerDay{
      font-size: 24px;
      margin-top: 10px;
    }
  }
  .tipDiv{
    border: 1px solid #e3e3e3;
    margin-top: 20px;
    border-radius: 20px;
    padding: 20px;
    font-size: 28px;
    overflow: hidden;
    .allApr{
      color: #000;
      .green{
        margin-left: 20px;
      }
    }
  }
  .marketReward{
    &>div{
      margin-top: 10px;
      &:first-child{
        margin-top: 0px;
      }
      &>span{
        line-height: 37px;
        &:first-child{
          margin-right: 10px;
        }
      }
      .small{
        font-size: 26px;
      }
      .changeImg{
        width: 30px;
      }
      .maxW{
        max-width: 480px;
      }
      .qusTip{
        padding: 0 0 0 10px;
        width: 30px;
        font-size: 24px;
      }
    }
  }
  .poolsLists{
    margin: 40px;
    .totalMiners{
      margin-right: 0;
      font-size: 26px;
      .tipIcon{
        margin-left: 10px;
        width: 28px;
      }
    }
    .list{
      margin-top: 25px;
      border: 1px solid #e0e0e0;
      border-radius: 20px;
      // padding: 20px;
      position: relative;
      &.page1{
        &:nth-child(2) {
          border: 1px solid rgb(238, 198, 4);
          box-shadow: 0 0 5px 0px rgba(238, 198, 4, .5);
        }
        &:nth-child(3) {
          border: 1px solid #b1dcff;
          box-shadow: 0 0 5px 0px rgba(#b1dcff, .5);
        }
        &:nth-child(4) {
          border: 1px solid #8C7853;
          box-shadow: 0 0 5px 0px rgba(#8C7853, .5);
        }
        .rankImg{
          position: absolute;
          top: -0px;
          left: -0px;
          width: 72px;
          transform: translate(-47%, -47%) rotate(-45deg);
          img{
            width: 100%;
          }
        }
      }
      &>div{
        padding: 20px;
        position: relative;
        z-index: 1;
        background: #FFF;
        border-radius: 30px;

        &:nth-child(1) {
          padding-bottom: 0;;
        }
        &:nth-child(2) {
          padding-top: 0;;
        }
      }
      .mb10{
        margin-bottom: 10px;
      }
    }
    .noData{
      text-align: center;
      margin: 100px 0;
      color: #A6A6A6;
      font-size: 24px;
    }
  }
}
.pagination{
  text-align: right;
  margin-top: 20px;
  font-size: 26px;
  /deep/ .el-pager{
    li.active{
      color: #07D79B;
    }
    li:hover{
      color: #07D79B;
    }
    li{
      font-size: 26px;
    }
  }
  /deep/ .btn-prev, /deep/ .btn-next{
    &:hover {
      color: #07D79B;
    }
    .el-icon-arrow-left, .el-icon-arrow-right{
      font-size: 26px;
    }
  }
}
.myDialog{
  /deep/ .el-dialog{
    position: relative;
    margin: auto;
    width: 570px;
    border-radius: 20px;
    .el-dialog__body,
    .el-dialog__header{
      padding: 0;
    }
  }
  &.apy{
    /deep/ .el-dialog{
      width: 620px;
    }
  }
}
</style>