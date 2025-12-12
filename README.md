# Digital Coupon Clippers

Automated JavaScript bookmarklets for clipping digital coupons from CVS and Publix. **Does not work on mobile browsers.**

### 🔗 Copy & Paste Bookmarklet Code

**CVS Clipper**: 
```javascript
javascript:(async()=>{const delay=ms=>new Promise(r=>setTimeout(r,ms)),scrollDelay=1000,clipDelay=500;let clipped=0,scrollCount=0;const isAtBottom=()=>(window.innerHeight+window.scrollY)>=document.body.offsetHeight-100;console.log("📜 CVS Clipper started...");while(!isAtBottom()&&scrollCount<50){let coupons=[...document.querySelectorAll("button.coupon-action.button-blue.sc-send-to-card-action")];for(let btn of coupons){if(btn.offsetParent!==null&&!btn.disabled){try{btn.scrollIntoView({behavior:"smooth",block:"center"});const rect=btn.getBoundingClientRect();const x=rect.left+rect.width/2+(Math.random()-0.5)*10;const y=rect.top+rect.height/2+(Math.random()-0.5)*10;btn.dispatchEvent(new MouseEvent('mousedown',{bubbles:true,clientX:x,clientY:y}));btn.dispatchEvent(new MouseEvent('mouseup',{bubbles:true,clientX:x,clientY:y}));btn.click();clipped++;console.log(`🔘 Clipped #${clipped}`);await delay(clipDelay);}catch(e){console.warn("⚠️ Clip failed:",e);}}}scrollCount++;window.scrollBy(0,500);await delay(scrollDelay);console.log(`📍 Scroll ${scrollCount}, at bottom: ${isAtBottom()}`);}console.log(`🎉 Done! Clipped ${clipped} coupons.`);alert(`🎉 Done!\nYou clipped ${clipped} new coupon${clipped===1?"":"s"}.`);})();
```

**Publix Clipper**: 
```javascript
javascript:(async()=>{console.log("🟢 Publix Clipper started");const delay=t=>new Promise(r=>setTimeout(r,t));const randomDelay=(min,max)=>delay(Math.floor(Math.random()*(max-min+1))+min);let clipped=0;const scrollToEl=e=>e.scrollIntoView({behavior:"smooth",block:"center"});const humanClick=async(btn)=>{const rect=btn.getBoundingClientRect();const x=rect.left+rect.width*0.5+Math.random()*10-5;const y=rect.top+rect.height*0.5+Math.random()*10-5;btn.focus();btn.dispatchEvent(new MouseEvent("mouseover",{bubbles:true,clientX:x,clientY:y}));await delay(50);btn.dispatchEvent(new MouseEvent("mousedown",{bubbles:true,clientX:x,clientY:y}));await delay(50);btn.dispatchEvent(new MouseEvent("mouseup",{bubbles:true,clientX:x,clientY:y}));btn.click();};async function clickLoadMoreAndWait(){const btn=document.querySelector('button[data-qa-automation="button-Load more"]');if(!btn)return false;console.log("⬇️ Load more found, clicking...");scrollToEl(btn);await humanClick(btn);window.scrollTo(0,document.body.scrollHeight);let retries=10,last=0;while(retries-->0){await delay(1000);let current=Array.from(document.querySelectorAll('button.p-coupon-button')).filter(e=>e.textContent?.trim()==="Clip coupon").length;if(current>last){console.log(`✅ Found ${current} coupons after load.`);return true;}}console.log("❌ No new coupons after Load more.");return false;}let pass=0,maxPasses=25;for(;pass++<maxPasses;){let buttons=Array.from(document.querySelectorAll('button.p-coupon-button')).filter(e=>e.textContent?.trim()==="Clip coupon");console.log(`🔍 Pass ${pass}: ${buttons.length} unclipped coupons`);if(buttons.length===0){const loaded=await clickLoadMoreAndWait();if(!loaded)break;}else{for(const btn of buttons){scrollToEl(btn);try{await humanClick(btn);clipped++;console.log(`🧾 Clipped #${clipped}`);await randomDelay(250,400);}catch(e){console.warn("❌ Failed to click:",e);}}}}console.log(`✅ Publix Clipper finished. Total clipped: ${clipped}`);alert(`✅ Publix Clipper finished. Total clipped: ${clipped}`);})();
```

**Kroger Clipper**: 
```javascript
javascript:(async()=>{console.log("🟢 Smart Kroger Clipper started");const delay=t=>new Promise(r=>setTimeout(r,t));const randomDelay=(min,max)=>delay(Math.floor(Math.random()*(max-min+1))+min);let clipped=0,unclipped=0;const MAX_COUPONS=250;const removedCoupons=[];const missedCoupons=[];const scrollToEl=e=>e.scrollIntoView({behavior:"smooth",block:"center"});const humanClick=async(btn)=>{const rect=btn.getBoundingClientRect();const x=rect.left+rect.width*0.5+Math.random()*10-5;const y=rect.top+rect.height*0.5+Math.random()*10-5;btn.focus();btn.dispatchEvent(new MouseEvent("mouseover",{bubbles:true,clientX:x,clientY:y}));await delay(50);btn.dispatchEvent(new MouseEvent("mousedown",{bubbles:true,clientX:x,clientY:y}));await delay(50);btn.dispatchEvent(new MouseEvent("mouseup",{bubbles:true,clientX:x,clientY:y}));btn.click();};const checkForMaximum=()=>{const headerText=document.body.innerText;return headerText.includes('You have reached the maximum number of offers')||headerText.includes('Maximum Number of Offers Clipped');};const isFuelPointsEvent=(couponElement)=>{const tags=couponElement.querySelectorAll('[role="note"]');return Array.from(tags).some(tag=>tag.getAttribute('aria-label')==='Fuel Points Event'||tag.innerText.includes('Fuel Points Event'));};const getCouponDescription=(couponElement)=>{const descEl=couponElement.querySelector('[data-testid^="CouponDesc-"]');return descEl?descEl.innerText.trim():'';};const getExpirationInfo=(couponElement)=>{const expDateEl=couponElement.querySelector('[data-testid^="CouponExpiryDate-"]');const expDaysEl=couponElement.querySelector('[data-testid^="CouponExpiryDays-"]');const dateText=expDateEl?expDateEl.innerText.replace('Exp.','').trim():'';const daysText=expDaysEl?expDaysEl.innerText.trim():'';return daysText||dateText;};const getExpirationDate=(couponElement)=>{const expiresText=couponElement.innerText;const match=expiresText.match(/Exp[.:]?\s*([A-Za-z]+\.?\s+\d+)/i);if(!match)return null;try{const expDate=new Date(match[1]+', '+new Date().getFullYear());if(expDate<new Date())expDate.setFullYear(expDate.getFullYear()+1);return expDate;}catch(e){return null;}};async function scrollAndWaitForMore(){const beforeScroll=document.body.scrollHeight;const beforeCount=document.querySelectorAll('[data-testid^="CouponActionButton-"]').length;window.scrollTo(0,document.body.scrollHeight);await delay(2000);const afterScroll=document.body.scrollHeight;const afterCount=document.querySelectorAll('[data-testid^="CouponActionButton-"]').length;return afterCount>beforeCount||afterScroll>beforeScroll;}console.log("📊 Step 1: Loading all available coupons...");let allLoaded=false,loadPass=0;while(!allLoaded&&loadPass++<100){const loaded=await scrollAndWaitForMore();if(!loaded)allLoaded=true;await delay(500);}console.log("✅ All coupons loaded");const allCoupons=Array.from(document.querySelectorAll('[data-testid^="CouponCard-"]')).map(coupon=>{const btn=coupon.querySelector('button[data-testid^="CouponActionButton-"]');if(!btn)return null;const isClipped=btn.textContent?.trim()==="Unclip";const isUnclipped=btn.textContent?.trim()==="Clip";const expDate=getExpirationDate(coupon);const isFuelPoints=isFuelPointsEvent(coupon);const description=getCouponDescription(coupon);const expirationInfo=getExpirationInfo(coupon);return{element:coupon,button:btn,isClipped,isUnclipped,isFuelPoints,description,expirationInfo,expirationDate:expDate,daysUntilExpire:expDate?(expDate-new Date())/(1000*60*60*24):999999};}).filter(c=>c!==null);const clippedCoupons=allCoupons.filter(c=>c.isClipped&&!c.isFuelPoints).sort((a,b)=>a.daysUntilExpire-b.daysUntilExpire);const protectedCoupons=allCoupons.filter(c=>c.isClipped&&c.isFuelPoints);const unclippedCoupons=allCoupons.filter(c=>c.isUnclipped);console.log(`📊 Status: ${clippedCoupons.length} clipped, ${protectedCoupons.length} fuel points (protected), ${unclippedCoupons.length} available`);const totalClipped=clippedCoupons.length+protectedCoupons.length;if(totalClipped>=MAX_COUPONS){const slotsNeeded=unclippedCoupons.length;const numToUnclip=Math.min(Math.max(slotsNeeded,50),clippedCoupons.length);console.log(`🧹 Step 2: Unclipping ${numToUnclip} oldest coupons to make room...`);for(let i=0;i<numToUnclip&&i<clippedCoupons.length;i++){const coupon=clippedCoupons[i];scrollToEl(coupon.button);await delay(100);try{await humanClick(coupon.button);unclipped++;removedCoupons.push({desc:coupon.description,exp:coupon.expirationInfo});console.log(`🗑️ Unclipped #${unclipped}: ${coupon.description}`);await randomDelay(200,400);}catch(e){console.warn("❌ Failed to unclip:",e);}}await delay(2000);console.log(`✅ Unclipped ${unclipped} old coupons, protected ${protectedCoupons.length} fuel points`);}console.log("✂️ Step 3: Clipping ALL new coupons...");window.scrollTo(0,0);await delay(1000);for(let i=0;i<unclippedCoupons.length;i++){if(checkForMaximum()){console.log("🛑 Hit maximum during clipping");for(let j=i;j<unclippedCoupons.length;j++){missedCoupons.push({desc:unclippedCoupons[j].description,exp:unclippedCoupons[j].expirationInfo});}break;}const coupon=unclippedCoupons[i];if(coupon.button.textContent?.trim()!=="Clip")continue;scrollToEl(coupon.button);await delay(100);try{await humanClick(coupon.button);clipped++;console.log(`🧾 Clipped #${clipped}: ${coupon.description}`);await randomDelay(300,500);if(checkForMaximum()){console.log("🛑 Reached 250 limit");for(let j=i+1;j<unclippedCoupons.length;j++){missedCoupons.push({desc:unclippedCoupons[j].description,exp:unclippedCoupons[j].expirationInfo});}break;}}catch(e){console.warn("❌ Failed to clip:",e);}}const finalCount=totalClipped-unclipped+clipped;console.log(`✅ Smart Kroger Clipper finished!`);let message=`✅ Kroger Clipper Done!\n\n🧾 Clipped: ${clipped} new\n🗑️ Removed: ${unclipped} old\n⛽ Protected: ${protectedCoupons.length} fuel pts\n📊 Total: ${finalCount}/250`;if(removedCoupons.length>0){message+=`\n\n📋 Removed (${removedCoupons.length}):\n`;removedCoupons.forEach(c=>{message+=`• ${c.desc} (${c.exp})\n`;});}if(missedCoupons.length>0){message+=`\n\n🚫 Couldn't Fit (${missedCoupons.length}):\n(Manually unclip others to add these)\n`;missedCoupons.forEach(c=>{message+=`• ${c.desc} (${c.exp})\n`;});}alert(message);console.log("📋 Full removed list:",removedCoupons);console.log("⚠️ Full missed list:",missedCoupons);})();

```

### Installation Instructions:
1. **Copy** the entire code block above (or appropriate .txt file)
2. **Right-click your bookmarks bar** → "Add page" or "Add bookmark"
3. **Name**: Enter the clipper name (i.e. "Publix Clipper")
4. **URL**: Paste the copied JavaScript code (starting with `javascript:`)
5. **Save the bookmark**

## How to Use

1. Navigate to the store's digital coupon page and sign in
2. Click your bookmarklet in the bookmarks bar
3. Watch the automation run (check browser console for progress)
4. Wait for completion alert showing total coupons clipped

## Specific to Kroger

1. Click the bookmarklet from your bookmarks bar
2. Wait 3-5 minutes while it loads and clips all coupons
3. Review the summary popup showing what was clipped, removed, and couldn't fit
4. Clips all available coupons up to Kroger's 250 limit
5. Protects Fuel Points Event coupons (never removes these)
6. Auto-removes oldest expiring coupons to make room for new ones
7. Shows detailed results: clipped, removed, and coupons that couldn't fit due to the 250 limit

## Legal Disclaimer

These bookmarklets are provided for educational and personal use only. Users are responsible for:

- Ensuring compliance with store Terms of Service
- Respecting website usage policies  
- Using automation responsibly and ethically

**Use at your own discretion and risk.**

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
