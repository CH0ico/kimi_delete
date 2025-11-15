async function sleep(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
}
 
async function handleDeleteFromMenu(menuButton, index) {
    try {
        console.log(`🔹 展开第 ${index + 1} 条会话的菜单`);
 
        // 如果是 SVG 或 span，找最近的可点击按钮
        const realButton = menuButton.closest('button, div, li');
        if (!realButton || typeof realButton.click !== 'function') {
            console.warn(`⚠️ 无法点击第 ${index + 1} 条菜单按钮，跳过`);
            return;
        }
 
        realButton.click();
        await sleep(100);
 
        const deleteOption = document.querySelector('li.opt-item.delete');
        if (!deleteOption) {
            console.warn("⚠️ 未找到删除按钮，跳过该条");
            return;
        }
 
        deleteOption.click();
        await sleep(100);
 
        const confirmBtn = document.querySelector('button.kimi-button.danger');
        if (confirmBtn) {
            confirmBtn.click();
            console.log(`🗑️ 已确认删除第 ${index + 1} 条`);
        } else {
            console.warn("⚠️ 未找到确认删除按钮");
        }
 
        await sleep(200);
    } catch (err) {
        console.error(`❌ 删除第 ${index + 1} 条时失败:`, err);
    }
}
 
async function main() {
    await sleep(200);
 
    const historyTrigger = Array.from(document.querySelectorAll('span'))
        .find(el => el.textContent.trim() === '历史会话');
 
    if (!historyTrigger) {
        console.warn("⚠️ 未找到“历史会话”按钮，终止操作");
        return;
    }
 
    historyTrigger.click();
    console.log("✅ 已点击历史会话按钮");
    await sleep(200);
 
    let attempts = 0;
    while (true) {
        const menuButtons = Array.from(document.querySelectorAll('.chat-item .opts-btn, .opts-icon, .more, svg[name="More"]')).filter(btn => btn.offsetParent !== null);
 
        if (menuButtons.length === 0) {
            console.log("✅ 没有可删除的历史记录，操作完成！");
            break;
        }
 
        console.log(`🔍 检测到 ${menuButtons.length} 个菜单按钮，开始逐个展开删除`);
 
        for (let i = 0; i < menuButtons.length; i++) {
            await handleDeleteFromMenu(menuButtons[i], i);
            await sleep(200);
        }
 
        console.log("🔁 重新获取列表以检查是否还有可删除项...");
        await sleep(500);
 
        attempts++;
        if (attempts > 20) {
            console.warn("⛔ 达到最大尝试次数，中止操作以避免死循环");
            break;
        }
    }
 
    console.log("🎉 全部历史会话应已删除完毕！");
}
 
setTimeout(main, 200);
