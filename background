var defaultRgx = ["<all_urls>", "*://*/*", "https://*.w3schools.com/*"].join('\n');
var theRegex = null;

// 需要删除/修改的安全头
var headersdo = {
    "x-frame-options": () => false,
    "content-security-policy": () => false,
    "x-content-type-options": () => false,
    "cross-origin-resource-policy": () => false
};

function updateRegexpes() {
    browser.storage.local.get(null, function(res) {
        var regstr = (res.regstr_allowed || defaultRgx);
        browser.webRequest.onHeadersReceived.removeListener(setHeader);

        if (!res.is_disabled) {
            theRegex = new RegExp(
                regstr.split("\n").map(
                    x => x.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')
                          .replace(/(^<all_urls>|\\\*)/g, "(.*?)")
                          .replace(/^(.*)$/g, "^$1$")
                ).join("|")
            );

            browser.webRequest.onHeadersReceived.addListener(
                setHeader,
                { urls: ["<all_urls>"], types: ["sub_frame", "object", "image", "media"] },
                ["blocking", "responseHeaders"]
            );
        }
    });
}

function setHeader(e) {
    return new Promise((resolve) => {
        (e.tabId === -1
            ? Promise.resolve({ url: e.originUrl })
            : browser.webNavigation.getFrame({ tabId: e.tabId, frameId: e.parentFrameId })
        ).then(parentFrame => {
            if (parentFrame.url.match(theRegex)) {
                e.responseHeaders = e.responseHeaders.filter(
                    x => (headersdo[x.name.toLowerCase()] || Array)()
                );
            }
            resolve({ responseHeaders: e.responseHeaders });
        });
    });
}

// 监听 popup 消息
var portFromCS;
function connected(p) {
    portFromCS = p;
    portFromCS.onMessage.addListener(function(m) {
        browser.storage.local.set(m, updateRegexpes);
    });
}

browser.runtime.onConnect.addListener(connected);
updateRegexpes();
console.log("Enhanced Ignore X-Frame-Options loaded");
