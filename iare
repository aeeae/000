// cdn缓存
const App = {
    plugins: [],
    
    use(plugin) {
        this.plugins.push(plugin);
        return this;
    },
    
    async init() {
        for (const plugin of this.plugins) {
            if (typeof plugin.init === 'function') {
                await plugin.init();
            }
        }
    }
};


App.use({
    name: 'BaseConfig',
    API_BASE: 'nav.php',
});


// 搜索
App.use({
    name: 'SearchPlugin',
    engines: {},
    currentCategory: 'changyong',
    selectedEngine: null,
    
    async init() {
        await this.loadEngines();
        this.initTimeDisplay();
        this.initCategoryTabs();
        this.initEnginesTabs();
        this.initSearchInput();
        this.initSearchButton();
        this.initSearchHistory(); 
    },
    
    async loadEngines() {
        try {
            const response = await fetch('search.php');
            if (!response.ok) {
                throw new Error(`HTTP error! status: ${response.status}`);
            }
            this.engines = await response.json();
        } catch (error) {
            console.error('Failed to load search engines:', error);
            this.engines = this.getDefaultEngines(); 
        }
    },
// 备用搜索配置    
    getDefaultEngines() {
        return {
            web: {
                baidu: { name: '百度', url: 'https://www.baidu.com/s?wd=' },
                bing: { name: '必应', url: 'https://cn.bing.com/search?q=' }
            }
        };
    },
    
initTimeDisplay() {
    const updateTime = () => {
        const now = new Date();
        const timeStr = now.toLocaleTimeString('zh-CN', { hour12: false });
    
        const year = now.getFullYear();
        const month = now.getMonth() + 1; 
        const day = now.getDate();
        const weekDay = ['星期日', '星期一', '星期二', '星期三', '星期四', '星期五', '星期六'][now.getDay()];
        const dateStr = `${year}年${month}月${day}日 ${weekDay}`;
        
        const timeMain = document.getElementById('timeMain');
        const timeDate = document.getElementById('timeDate');
        
        if (timeMain) timeMain.textContent = timeStr;
        if (timeDate) timeDate.textContent = dateStr;
    };
    updateTime();
    setInterval(updateTime, 1000);
},

    
    initCategoryTabs() {
        const tabs = document.querySelectorAll('.category-tab');
        if (tabs.length === 0) return;
        
        tabs.forEach(tab => {
            tab.addEventListener('click', () => {
                tabs.forEach(t => t.classList.remove('active'));
                tab.classList.add('active');
                this.currentCategory = tab.dataset.category;
                this.initEnginesTabs();
            });
        });
    },
    
    initEnginesTabs() {
        const container = document.getElementById('enginesTabs');
        if (!container) return;
        
        container.innerHTML = '';
        
        const categoryEngines = this.engines[this.currentCategory] || {};
        this.selectedEngine = null;
        
        Object.entries(categoryEngines).forEach(([key, engine], index) => {
            const tab = document.createElement('button');
            tab.className = 'engine-tab';
            if (index === 0) {
                tab.classList.add('active');
                this.selectedEngine = key;
            }
            tab.dataset.engine = key;
            tab.innerHTML = `
                <svg viewBox="0 0 24 24" fill="none" stroke="currentColor">
                    <circle cx="12" cy="12" r="10"></circle>
                    <path d="M12 6v6l4 2"></path>
                </svg>
                ${engine.name}
            `;
            
            tab.addEventListener('click', () => {
                container.querySelectorAll('.engine-tab').forEach(t => t.classList.remove('active'));
                tab.classList.add('active');
                this.selectedEngine = key;
            });
            
            container.appendChild(tab);
        });
    },
    
    initSearchInput() {
        const input = document.getElementById('searchInput');
        const clearBtn = document.getElementById('clearBtn');
        const dropdownPanel = document.getElementById('dropdownPanel');
        
        if (!input || !clearBtn || !dropdownPanel) return;
        
        const suggestionsSection = document.getElementById('suggestionsSection');
        const historySection = document.getElementById('historySection');
        let suggestionTimeout;
        
        input.addEventListener('input', async (e) => {
            const query = e.target.value.trim();
            clearBtn.style.display = query ? 'block' : 'none';
            
            if (query || this.getSearchHistory().length > 0) {
                dropdownPanel.style.display = 'block';
                
                if (document.getElementById('suggestionsToggle')?.checked && query) {
                    clearTimeout(suggestionTimeout);
                    suggestionTimeout = setTimeout(() => {
                        this.fetchSuggestions(query);
                    }, 300);
                } else {
                    this.clearSuggestions();
                }
                
                if (document.getElementById('historyToggle')?.checked) {
                    this.displaySearchHistory();
                }
            } else {
                dropdownPanel.style.display = 'none';
            }
        });
        
        input.addEventListener('focus', () => {
            if (input.value.trim() || this.getSearchHistory().length > 0) {
                dropdownPanel.style.display = 'block';
                if (document.getElementById('historyToggle')?.checked) {
                    this.displaySearchHistory();
                }
            }
        });
        
        clearBtn.addEventListener('click', () => {
            input.value = '';
            clearBtn.style.display = 'none';
            dropdownPanel.style.display = 'none';
            input.focus();
        });
        
        document.addEventListener('click', (e) => {
            if (!e.target.closest('.search-box-wrapper')) {
                dropdownPanel.style.display = 'none';
            }
        });
        
        this.initToggleEvents();
        
        input.addEventListener('keypress', (e) => {
            if (e.key === 'Enter') {
                this.performSearch();
            }
        });
    const searchBtnInline = document.getElementById('searchBtnInline');
    if (searchBtnInline) {
        searchBtnInline.addEventListener('click', () => {
            this.performSearch();
        });
    }
    },
    
initToggleEvents() {
    const suggestionsToggle = document.getElementById('suggestionsToggle');
    const historyToggle = document.getElementById('historyToggle');

    if (suggestionsToggle) {
        const savedSuggestionsState = localStorage.getItem('searchSuggestionsEnabled');
        suggestionsToggle.checked = savedSuggestionsState !== null ? savedSuggestionsState === 'true' : true;
        suggestionsToggle.addEventListener('change', (e) => {
            const isEnabled = e.target.checked;
            localStorage.setItem('searchSuggestionsEnabled', isEnabled);
            
            const input = document.getElementById('searchInput');
            const dropdownPanel = document.getElementById('dropdownPanel');
            if (input && dropdownPanel) {
                const query = input.value.trim();
                if (query && isEnabled) {
                    this.fetchSuggestions(query);
                } else {
                    this.clearSuggestions();
                }

                if (!isEnabled && !historyToggle.checked && !query) {
                    dropdownPanel.style.display = 'none';
                }
            }
        });
    }

    if (historyToggle) {
        const savedHistoryState = localStorage.getItem('searchHistoryEnabled');
        historyToggle.checked = savedHistoryState !== null ? savedHistoryState === 'true' : true;

        historyToggle.addEventListener('change', (e) => {
            const isEnabled = e.target.checked;
            localStorage.setItem('searchHistoryEnabled', isEnabled);

            const input = document.getElementById('searchInput');
            const dropdownPanel = document.getElementById('dropdownPanel');
            if (input && dropdownPanel) {
                if (isEnabled) {
                    this.displaySearchHistory();
                } else {
                    this.clearHistory();
                }
                if (!isEnabled && !suggestionsToggle.checked && !input.value.trim()) {
                    dropdownPanel.style.display = 'none';
                }
            }
        });
    }
},

    
    fetchSuggestions(query) {
        const suggestionsSection = document.getElementById('suggestionsSection');
        if (!suggestionsSection) return;
        
        try {
            const callbackName = 'search_callback_' + Date.now();
            const script = document.createElement('script');
            script.src = `https://www.baidu.com/su?&wd=${encodeURIComponent(query)}&cb=${callbackName}`;
            
            window[callbackName] = (data) => {
                if (data.s && data.s.length > 0) {
                    const suggestionsList = suggestionsSection.querySelector('.suggestions-list');
                    if (suggestionsList) {
                        suggestionsList.innerHTML = data.s.map(s => `
                            <div class="suggestion-item">${s}</div>
                        `).join('');
                        
                        suggestionsList.querySelectorAll('.suggestion-item').forEach(item => {
                            item.addEventListener('click', () => {
                                document.getElementById('searchInput').value = item.textContent.trim();
                                this.performSearch();
                            });
                        });
                    }
                } else {
                    this.clearSuggestions();
                }
                
                this.cleanupScript(script, callbackName);
            };
            
            script.onerror = () => {
                this.clearSuggestions();
                this.cleanupScript(script, callbackName);
            };
            
            document.head.appendChild(script);

            setTimeout(() => {
                if (window[callbackName]) {
                    this.clearSuggestions();
                    this.cleanupScript(script, callbackName);
                }
            }, 5000);
            
        } catch (error) {
            console.error('Failed to fetch suggestions:', error);
            this.clearSuggestions();
        }
    },
    
    cleanupScript(script, callbackName) {
        if (document.head.contains(script)) {
            document.head.removeChild(script);
        }
        if (window[callbackName]) {
            delete window[callbackName];
        }
    },
    
    clearSuggestions() {
        const suggestionsSection = document.getElementById('suggestionsSection');
        if (suggestionsSection) {
            const suggestionsList = suggestionsSection.querySelector('.suggestions-list');
            if (suggestionsList) suggestionsList.innerHTML = '';
        }
    },
    
    initSearchButton() {
        const searchBtn = document.getElementById('searchBtn');
        const clearHistoryBtn = document.getElementById('clearHistory');
        
        if (searchBtn) {
            searchBtn.addEventListener('click', () => {
                this.performSearch();
            });
        }
        
        if (clearHistoryBtn) {
            clearHistoryBtn.addEventListener('click', () => {
                localStorage.removeItem('searchHistory');
                this.displaySearchHistory();
            });
        }
    },
    
    initSearchHistory() {
        this.loadSearchHistory();
    },
    
    loadSearchHistory() {
        this.displaySearchHistory();
    },
    
    performSearch() {
        const input = document.getElementById('searchInput');
        if (!input) return;
        
        const query = input.value.trim();
        
        if (!query) {
            input.focus();
            return;
        }
        
        if (!this.selectedEngine) {
            const engines = Object.keys(this.engines[this.currentCategory] || {});
            if (engines.length > 0) {
                this.selectedEngine = engines[0];
            }
        }
        
        this.saveSearchHistory(query);
        
        const engine = this.engines[this.currentCategory]?.[this.selectedEngine];
        if (engine) {
            const url = engine.url + encodeURIComponent(query);
            window.open(url, '_blank');
        }
        
        const dropdownPanel = document.getElementById('dropdownPanel');
        if (dropdownPanel) dropdownPanel.style.display = 'none';
    },
    
    saveSearchHistory(query) {
        if (!document.getElementById('historyToggle')?.checked) return;
        
        try {
            let history = this.getSearchHistory();
            history = history.filter(item => item !== query);
            history.unshift(query);
            history = history.slice(0, 10);
            
            localStorage.setItem('searchHistory', JSON.stringify(history));
        } catch (error) {
            console.error('Failed to save search history:', error);
        }
    },
    
    getSearchHistory() {
        try {
            return JSON.parse(localStorage.getItem('searchHistory') || '[]');
        } catch {
            return [];
        }
    },
    
    displaySearchHistory() {
        const history = this.getSearchHistory();
        const historySection = document.getElementById('historySection');
        if (!historySection) return;
        
        const historyList = historySection.querySelector('.history-list');
        if (!historyList) return;
        
        if (history.length === 0) {
            historyList.innerHTML = '<div class="history-item">暂无搜索历史</div>';
            return;
        }
        
        historyList.innerHTML = history.map(item => `
            <div class="history-item">${item}</div>
        `).join('');
        
        historyList.querySelectorAll('.history-item').forEach(item => {
            item.addEventListener('click', () => {
                const input = document.getElementById('searchInput');
                if (input) {
                    input.value = item.textContent;
                    this.performSearch();
                }
            });
        });
    }
});


// 导航
App.use({
    name: 'NavPlugin',
    icons: ['✨', '📖', '🎦', '🤖', '🎒', '💬', '📚','☘️','⛱️','🏄️','💹','📱','🚗','🎨','💻','✍️','🎈'],
    
    async init() {
        await this.loadFeaturedCategories();
        this.setupLoadMore();
    },
    
    async loadFeaturedCategories() {
        const container = document.getElementById('navCategories');
        if (!container) return;

        try {
            const response = await fetch(`${App.plugins[0].API_BASE}?action=featured`);
            const categories = await response.json();
            
            let html = categories.map((category, index) => 
                this.renderCategory(category, index, true)
            ).join('');
            
            html += `
                <div class="load-more-container" id="loadMoreContainer">
                    <button class="load-more-btn" id="loadMoreBtn">
                        查看全部
                    </button>
                </div>
            `;
            
            container.innerHTML = html;
            this.setupLoadMore();
        } catch (error) {
            console.error('加载精选导航失败:', error);
            container.innerHTML = '<p style="text-align: center; color: #65676b;">加载失败，请刷新重试</p>';
        }
    },
    
async loadAllCategories() {
    const container = document.getElementById('navCategories');
    if (!container) return;
    
    try {
        const loadMoreBtn = document.getElementById('loadMoreBtn');
        loadMoreBtn.innerHTML = '加载中...';
        loadMoreBtn.disabled = true;
        const response = await fetch(`${App.plugins[0].API_BASE}?action=all`);
        const categories = await response.json();
        const otherCategories = categories.filter(cat => !cat.featured);
        const loadMoreContainer = document.getElementById('loadMoreContainer');
        const otherCategoriesHtml = otherCategories.map((category, index) => 
            this.renderCategory(category, index + 1, false)
        ).join('');
        
        loadMoreContainer.insertAdjacentHTML('beforebegin', otherCategoriesHtml);
        loadMoreContainer.style.display = 'none';
    } catch (error) {
        console.error('加载全部导航失败:', error);
        const loadMoreBtn = document.getElementById('loadMoreBtn');
        loadMoreBtn.innerHTML = '<span class="icon">⬇️</span> 加载全部导航';
        loadMoreBtn.disabled = false;
        
        alert('加载失败，请重试');
    }
},
    
    renderCategory(category, index, isFeatured = false) {
        const icon = this.icons[index] || '📁';
        
        return `
            <div class="category">
                <div class="category-header">
                    <div class="category-icon">${icon}</div>
                    <h3 class="category-title">${category.title}</h3>
                </div>
                <div class="nav-links">
                    ${category.links.map(link => `
                        ${isFeatured && link.icon ? 
                            `<a href="${link.url}" target="_blank" class="featured-link">
                                <img src="${link.icon}" alt="${link.name}" class="featured-icon" onerror="this.style.display='none'">
                                ${link.name}
                            </a>` :
                            `<a href="${link.url}" target="_blank" class="nav-link">${link.name}</a>`
                        }
                    `).join('')}
                </div>
            </div>
        `;
    },
    
setupLoadMore() {
    const loadMoreBtn = document.getElementById('loadMoreBtn');
    if (loadMoreBtn) {
        loadMoreBtn.replaceWith(loadMoreBtn.cloneNode(true));
        const newLoadMoreBtn = document.getElementById('loadMoreBtn');
        newLoadMoreBtn.addEventListener('click', () => {
            this.loadAllCategories();
        });
    }
}
});


// 资讯流
App.use({
    name: 'NewsFeedPlugin',
    currentPage: 0,
    pageSize: 20,
    allData: [],
    hasMore: true,
    
    async init() {
        await this.loadNewsFeed();
    },
    
    async loadNewsFeed() {
        const container = document.getElementById('newsFeedList');
        if (!container) return;
        
        try {
            this._showLoading(container, '加载中...');
            
            const response = await fetch('news.php');
            const data = await response.json();
            
            if (data.success && data.data) {
                this.allData = data.data.cards[0].content || [];
                this.currentPage = 0;
                this.hasMore = this.allData.length > this.pageSize;
                
                this.renderNewsFeed();
                this.updateLastUpdateTime();
                
                this.updateLoadMoreButton();
            } else {
                throw new Error(data.message || '数据格式错误');
            }
        } catch (error) {
            console.error('加载资讯流失败:', error);
            this._showError(container, '加载失败，请稍后重试', () => this.loadNewsFeed());
        }
    },
    
    async loadMoreNews() {
        if (!this.hasMore) return;
        const loadMoreBtn = document.querySelector('.news-feed-container #newsFeedMoreBtn');
        
        if (loadMoreBtn) {
            loadMoreBtn.disabled = true;
            loadMoreBtn.textContent = '加载中...';
        }
        
        try {
            this.currentPage++;
            this.renderNewsFeed(true); 

            const startIndex = (this.currentPage + 1) * this.pageSize;
            this.hasMore = startIndex < this.allData.length;
            
            this.updateLoadMoreButton();
        } catch (error) {
            console.error('加载更多失败:', error);
            this.showToast('加载更多失败，请重试');
            
            if (loadMoreBtn) {
                loadMoreBtn.disabled = false;
                loadMoreBtn.textContent = '加载更多';
            }
        }
    },
    
    updateLoadMoreButton() {
        const footer = document.getElementById('newsFeedFooter');
        const loadMoreBtn = document.querySelector('.news-feed-container #newsFeedMoreBtn');
        
        if (!footer || !loadMoreBtn) return;
        
        if (this.allData.length === 0) {
            footer.style.display = 'none';
            return;
        }
        
        footer.style.display = 'block';
        
        if (this.hasMore) {
            loadMoreBtn.textContent = '加载更多';
            loadMoreBtn.disabled = false;
            loadMoreBtn.classList.remove('completed');
        } else {
            loadMoreBtn.textContent = '已加载全部';
            loadMoreBtn.disabled = true;
            loadMoreBtn.classList.add('completed');
        }
    },
    
    async refreshNews() {
        const refreshBtn = document.querySelector('.news-feed-container .refresh-btn-header');
        const refreshIcon = refreshBtn ? refreshBtn.querySelector('.refresh-icon') : null;
        
        if (refreshBtn) {
            refreshBtn.disabled = true;
        }
        if (refreshIcon) {
            refreshIcon.classList.add('spinning');
        }
        
        try {
            const container = document.getElementById('newsFeedList');
            this._showLoading(container, '正在刷新...');

            const response = await fetch('news.php?force=1');
            const data = await response.json();

            if (data.success && data.data) {
                this.allData = data.data.cards[0].content || [];
                this.currentPage = 0;
                this.hasMore = this.allData.length > this.pageSize;
                
                this.renderNewsFeed();
                this.updateLastUpdateTime();
                this.showToast('刷新成功');
                this.updateLoadMoreButton();
            } else {
                throw new Error(data.message || '刷新失败，请检查网络连接');
            }
        } catch (error) {
            console.error('刷新失败:', error);
            this.showToast(error.message);
        } finally {
            if (refreshBtn) {
                refreshBtn.disabled = false;
            }
            if (refreshIcon) {
                refreshIcon.classList.remove('spinning');
            }
        }
    },
    
    renderNewsFeed(append = false) {
        const container = document.getElementById('newsFeedList');
        if (!container) return;
        
        const startIndex = this.currentPage * this.pageSize;
        const endIndex = Math.min(startIndex + this.pageSize, this.allData.length);
        const currentData = this.allData.slice(startIndex, endIndex);
        
        const newsItems = currentData.map(item => `
            <div class="news-item" data-index="${item.index}">
                <div class="news-rank">${item.index + 1}</div>
                <div class="news-content">
                    <div class="news-header">
                        <h4 class="news-title">
                            <a href="${item.rawUrl.replace('m.baidu.com', 'www.baidu.com')}" target="_blank" rel="noopener noreferrer">${item.word}</a>
                        </h4>
                        ${item.hotTagImg ? `<img src="${item.hotTagImg}" alt="热点标签" class="hot-tag-img">` : ''}
                    </div>
                    ${item.desc ? `<p class="news-desc">${item.desc}</p>` : ''}
                    <div class="news-meta">
                        <span class="hot-score">${this.formatHotScore(item.hotScore)}</span>
                        ${item.hotChange !== 'same' ? `<span class="hot-change ${item.hotChange}">${this.getHotChangeText(item.hotChange)}</span>` : ''}
                    </div>
                </div>
                ${item.img ? `<div class="news-image"><img src="${item.img}" alt="${item.word}" loading="lazy"></div>` : ''}
            </div>
        `).join('');
        
        if (append) {
            container.insertAdjacentHTML('beforeend', newsItems);
        } else {
            container.innerHTML = newsItems;
        }
    },
    
    formatHotScore(score) {
        const num = parseInt(score, 10);
        if (num >= 10000000) return (num / 10000000).toFixed(1) + '千万';
        if (num >= 10000) return (num / 10000).toFixed(1) + '万';
        return num.toString();
    },
    
    getHotChangeText(change) {
        const changes = {
            'up': '↑',
            'down': '↓',
            'new': '新'
        };
        return changes[change] || '';
    },
    
    updateLastUpdateTime() {
        const updateElement = document.querySelector('.news-feed-container .last-update');
        if (updateElement) {
            const now = new Date();
            updateElement.textContent = `最后更新: ${now.getHours().toString().padStart(2, '0')}:${now.getMinutes().toString().padStart(2, '0')}`;
        }
    },
    
    showToast(message) {
        const linksPlugin = App.plugins.find(p => p.name === 'LinksPlugin');
        if (linksPlugin && typeof linksPlugin.showToast === 'function') {
            linksPlugin.showToast(message);
            return;
        }
        
        const toast = document.createElement('div');
        toast.className = 'toast';
        toast.textContent = message;
        document.body.appendChild(toast);
        
        setTimeout(() => {
            toast.style.animation = 'toastIn 0.3s ease reverse';
            setTimeout(() => toast.remove(), 300);
        }, 2000);
    },

    _showLoading(container, message) {
        container.innerHTML = `
            <div class="loading-spinner">
                <div class="spinner"></div>
                <p>${message}</p>
            </div>
        `;
    },

    _showError(container, message, retryCallback) {
        container.innerHTML = `
            <div class="error-message">
                <i class="error-icon"></i>
                <p>${message}</p>
                <button onclick="(${retryCallback.toString()})()">重试</button>
            </div>
        `;
    }
});


// 收藏
App.use({
    name: 'LinksPlugin',
    
    init() {
        this.loadCustomLinks();
        setTimeout(() => this.bindEvents(), 100);
    },
    
    bindEvents() {
        const addBtn = document.querySelector('.add-btn');
        if (addBtn) {
            addBtn.replaceWith(addBtn.cloneNode(true));
            const newAddBtn = document.querySelector('.add-btn');
            newAddBtn.addEventListener('click', (e) => {
                e.preventDefault();
                this.addCustomLink();
            });
        } else {
            console.log('未找到添加按钮');
        }
        
        const customName = document.getElementById('customName');
        const customUrl = document.getElementById('customUrl');
        
        if (customName) {
            customName.addEventListener('keypress', (e) => {
                if (e.key === 'Enter') this.addCustomLink();
            });
        }
        
        if (customUrl) {
            customUrl.addEventListener('keypress', (e) => {
                if (e.key === 'Enter') this.addCustomLink();
            });
        }
    },
    
    addCustomLink() {
        console.log('addCustomLink 被调用');
        const name = document.getElementById('customName').value.trim();
        const url = document.getElementById('customUrl').value.trim();
        
        if (!name || !url) {
            this.showToast('请填写完整的网站名称和网址');
            return;
        }
        
        if (!url.startsWith('http://') && !url.startsWith('https://')) {
            this.showToast('请输入有效的网址（以http://或https://开头）');
            return;
        }
        
        let links = JSON.parse(localStorage.getItem('customLinks') || '[]');
        
        if (links.some(link => link.url === url)) {
            this.showToast('该网址已存在收藏中');
            return;
        }
        
        links.push({ 
            name, 
            url, 
            id: Date.now(),
            createdAt: new Date().toISOString()
        });
        
        localStorage.setItem('customLinks', JSON.stringify(links));
        
        document.getElementById('customName').value = '';
        document.getElementById('customUrl').value = '';
        
        this.loadCustomLinks();
        this.showToast('添加成功');
    },
    
    loadCustomLinks() {
        const links = JSON.parse(localStorage.getItem('customLinks') || '[]');
        const container = document.getElementById('savedLinks');
        if (!container) return;
        
        if (links.length === 0) {
            container.innerHTML = '<p style="text-align: center; color: #65676b; font-size: 0.875rem;">暂无</p>';
            return;
        }
        
        const sortedLinks = links.sort((a, b) => new Date(b.createdAt) - new Date(a.createdAt));
        
        container.innerHTML = sortedLinks.map(link => `
            <div class="saved-link">
                <a href="${link.url}" target="_blank" class="saved-link-item">${link.name}</a>
                <button class="delete-btn" onclick="App.plugins.find(p => p.name === 'LinksPlugin').deleteCustomLink(${link.id})">删除</button>
            </div>
        `).join('');
    },
    
    deleteCustomLink(id) {
        let links = JSON.parse(localStorage.getItem('customLinks') || '[]');
        links = links.filter(link => link.id !== id);
        localStorage.setItem('customLinks', JSON.stringify(links));
        this.loadCustomLinks();
        this.showToast('已删除');
    },
    
    showToast(message) {
        const toast = document.createElement('div');
        toast.className = 'toast';
        toast.textContent = message;
        document.body.appendChild(toast);
        
        setTimeout(() => {
            toast.style.animation = 'toastIn 0.3s ease reverse';
            setTimeout(() => toast.remove(), 300);
        }, 2000);
    }
});


// 位置天气
App.use({
    name: 'CombinedInfoPlugin',
    
    async init() {
        await this.loadCombinedInfo();
        this.startTimeUpdate();
    },
    
    async loadCombinedInfo() {
        const container = document.getElementById('combinedInfo');
        if (!container) return;

        try {
            container.innerHTML = `
                <div style="text-align: center; padding: 40px 20px;">
                    <div class="spinner"></div>
                </div>
            `;
            
            const response = await fetch('ip.php');
            if (!response.ok) {
                throw new Error(`网络请求失败: ${response.statusText}`);
            }
            const data = await response.json();
            
            if (data.status !== 'success') {
                throw new Error('获取数据失败');
            }

            this.renderCombinedInfo(container, data);
        } catch (error) {
            console.error('加载综合信息失败:', error);
            container.innerHTML = `
                <div class="error-message">
                    <p style="text-align: center; color: #65676b; font-size: 0.875rem;">
                        信息加载失败: ${error.message}
                    </p>
                </div>
            `;
        }
    },
    
    renderCombinedInfo(container, data) {
        const { location, weather, time, ip } = data;
        const currentTime = this.getCurrentTime();
        
        container.innerHTML = `
            <div class="combined-info">
                <div class="info-header">
                    <div class="location-section">
                        <div class="location-details">
                            <div class="main-location">📍${location.city}</div>
                            <div class="sub-location">${location.province} • IP: ${ip}</div>
                        </div>
                    </div>
                </div>
                <div class="weather-section">
                    <div class="current-weather-main">
                        <div class="weather-left">
                            <div class="temperature">${weather.current.temperature}°</div>
                            <div class="weather-condition">${weather.current.weather}</div>
                        </div>
                        <div class="weather-right">
                            <div class="weather-detail">
                                <span class="detail-label">湿度</span>
                                <span class="detail-value">${weather.current.humidity}%</span>
                            </div>
                            <div class="weather-detail">
                                <span class="detail-label">风向</span>
                                <span class="detail-value">${weather.current.winddirection}</span>
                            </div>
                            <div class="weather-detail">
                                <span class="detail-label">风力</span>
                                <span class="detail-value">${weather.current.windpower}级</span>
                            </div>
                        </div>
                    </div>
                    <div class="tomorrow-weather">
                        <div class="tomorrow-label">${weather.tomorrow.date_label}</div>
                        <div class="tomorrow-details">
                            <span class="tomorrow-weather-text">${weather.tomorrow.dayweather}</span>
                            <span class="tomorrow-temp">${weather.tomorrow.nighttemp}° ~ ${weather.tomorrow.daytemp}°</span>
                        </div>
                    </div>
                </div>
                <div class="info-footer">
                    <div class="update-time">
                        更新于 ${this.formatTime(weather.current.reporttime)}
                    </div>
                </div>
            </div>
        `;
    },
    
    startTimeUpdate() {
        setInterval(() => {
            const timeElement = document.getElementById('liveTime');
            if (timeElement) {
                timeElement.textContent = this.getCurrentTime();
            }
        }, 1000);
    },
    
    getCurrentTime() {
        const now = new Date();
        const hours = now.getHours().toString().padStart(2, '0');
        const minutes = now.getMinutes().toString().padStart(2, '0');
        const seconds = now.getSeconds().toString().padStart(2, '0');
        return `${hours}:${minutes}:${seconds}`;
    },
    
    formatTime(timeString) {
        return timeString.split(' ')[1].substring(0, 5);
    }
});


// 便签
App.use({
    name: 'NotesPlugin',
    autoSaveTimers: new Map(),
    
    init() {
        this.loadNotes();
        this.bindEvents();
    },
    
    bindEvents() {
        const addBtn = document.getElementById('addNoteBtn');
        if (addBtn) {
            addBtn.addEventListener('click', () => this.addNote());
        }
    },
    
    hasEmptyNote(notes) {
        return notes.some(note => !note.content || note.content.trim() === '');
    },
    
    addNote() {
        const notes = this.getNotesFromStorage();
        if (this.hasEmptyNote(notes)) {
            this.showToast('已经有个空白便签啦~');
            return; 
        }
        const newNote = {
            id: Date.now(),
            content: '',
            createdAt: new Date().toISOString(),
            updatedAt: new Date().toISOString()
        };
        
        notes.unshift(newNote);
        this.saveNotesToStorage(notes);
        this.renderNotes(notes);
        setTimeout(() => {
            const newNoteElement = document.querySelector(`[data-note-id="${newNote.id}"]`);
            if (newNoteElement) {
                newNoteElement.focus();
            }
        }, 0);
    },
    
    setupAutoSave(noteId, textarea) {
        const statusElement = document.querySelector(`[data-note-status-id="${noteId}"]`);
        if (statusElement) {
            statusElement.textContent = '编辑中...';
            statusElement.className = 'note-status editing'; 
        }
        if (this.autoSaveTimers.has(noteId)) {
            clearTimeout(this.autoSaveTimers.get(noteId));
        }
        
        const timer = setTimeout(() => {
            this.updateNote(noteId, textarea.value);
        }, 500); 
        
        this.autoSaveTimers.set(noteId, timer);
    },
    
    cleanupAutoSave(noteId) {
        if (this.autoSaveTimers.has(noteId)) {
            clearTimeout(this.autoSaveTimers.get(noteId));
            this.autoSaveTimers.delete(noteId);
        }
    },
    
    updateNote(id, content) {
        const notes = this.getNotesFromStorage();
        const noteIndex = notes.findIndex(note => note.id === id);
        
        if (noteIndex !== -1) {
            notes[noteIndex].content = content;
            notes[noteIndex].updatedAt = new Date().toISOString();
            this.saveNotesToStorage(notes);
            const statusElement = document.querySelector(`[data-note-status-id="${id}"]`);
            if (statusElement) {
                statusElement.textContent = '已保存';
                statusElement.className = 'note-status saved'; 
            }
        }
    },
    
    deleteNote(id) {
        this.cleanupAutoSave(id);
        
        const notes = this.getNotesFromStorage();
        const filteredNotes = notes.filter(note => note.id !== id);
        this.saveNotesToStorage(filteredNotes);
        this.renderNotes(filteredNotes);
        this.showToast('便签已删除');
    },
    
    getNotesFromStorage() {
        return JSON.parse(localStorage.getItem('userNotes') || '[]');
    },
    
    saveNotesToStorage(notes) {
        localStorage.setItem('userNotes', JSON.stringify(notes));
    },
    
    loadNotes() {
        const notes = this.getNotesFromStorage();
        this.renderNotes(notes);
    },
    
    renderNotes(notes) {
        const container = document.getElementById('notesList');
        if (!container) return;
        
        if (notes.length === 0) {
            container.innerHTML = '<p style="text-align: center; color: #65676b; font-size: 0.875rem;">暂无</p>';
            return;
        }
        
        container.innerHTML = notes.map(note => `
            <div class="note-item">
                <textarea 
                    class="note-content"
                    data-note-id="${note.id}"
                    placeholder="点击输入内容..."
                    oninput="App.plugins.find(p => p.name === 'NotesPlugin').setupAutoSave(${note.id}, this)"
                >${note.content}</textarea>
                <div class="note-footer">
                    <div class="note-info">
                        <!-- 【新增】状态显示区域 -->
                        <span class="note-status saved" data-note-status-id="${note.id}" style="font-size: 11px; color: #1877f2;">已保存</span>
                        <span class="note-time">${this.formatTime(note.createdAt)}</span>
                    </div>
                    <button class="note-delete-btn" onclick="App.plugins.find(p => p.name === 'NotesPlugin').deleteNote(${note.id})">删除</button>
                </div>
            </div>
        `).join('');
    },
    
    formatTime(isoString) {
        const date = new Date(isoString);
        const year = date.getFullYear();
        const month = String(date.getMonth() + 1).padStart(2, '0');
        const day = String(date.getDate()).padStart(2, '0');
        const hours = String(date.getHours()).padStart(2, '0');
        const minutes = String(date.getMinutes()).padStart(2, '0');
        
        return `${year}年${month}月${day}日 ${hours}:${minutes}`;
    },
    
    showToast(message) {
        const toast = document.createElement('div');
        toast.className = 'toast';
        toast.textContent = message;
        document.body.appendChild(toast);
        
        setTimeout(() => {
            toast.style.animation = 'toastIn 0.3s ease reverse';
            setTimeout(() => toast.remove(), 300);
        }, 2000);
    }
});



// 热榜
App.use({
    name: 'HotPlugin',
    currentPlatform: '今日头条', 
    
    async init() {
        this.bindEvents();
        await this.fetchHotData();
    },
    
    bindEvents() {
        document.querySelectorAll('.tab-btn').forEach(btn => {
            btn.addEventListener('click', (e) => {
                document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
                e.target.classList.add('active');
                this.currentPlatform = e.target.dataset.platform;
                this.fetchHotData();
            });
        });
    },
    
    async fetchHotData() {
        const hotList = document.getElementById('hotList');
        if (!hotList) return;
        
        try {
            hotList.innerHTML = `
                <div style="text-align: center; padding: 20px;">
                    <div class="spinner"></div>
                </div>
            `;

            const response = await fetch(`hot.php?title=${encodeURIComponent(this.currentPlatform)}`);
            const result = await response.json();

            if (result.code === 200 && result.data) {
                this.renderHotList(result.data);
            } else {
                throw new Error(result.message || '数据获取失败');
            }
        } catch (error) {
            console.error('获取热榜数据失败:', error);
            this.showError(hotList);
        }
    },
    
renderHotList(data) {
    const hotList = document.getElementById('hotList');
    if (!hotList) return;
    
    if (!data || data.length === 0) {
        hotList.innerHTML = '<div style="text-align: center; padding: 20px; color: #666;">暂无数据</div>';
        return;
    }

    const top15Data = data.slice(0, 15);
    
    let html = '';
    top15Data.forEach((item, index) => {
        const rankClass = index < 3 ? `rank-number top-${index + 1}` : 'rank-number';
        let displayTitle = item.title;
        if (item.title && item.title.length > 16) {
            displayTitle = item.title.substring(0, 16) + '...';
        }
        
        html += `
            <div class="hot-item">
                <div class="hot-rank">
                    <span class="${rankClass}">${index + 1}</span>
                </div>
                <div class="hot-content">
                    <a href="${item.url}" target="_blank" class="hot-title" title="${item.title}">
                        ${displayTitle}
                    </a>
                </div>
            </div>
        `;
    });

    hotList.innerHTML = html;
},

    
    showError(container) {
        container.innerHTML = `
            <div style="text-align: center; padding: 20px; color: #666;">
                <p>加载失败，请稍后重试</p>
                <button onclick="App.plugins.find(p => p.name === 'HotPlugin').fetchHotData()" style="margin-top: 10px; padding: 5px 15px; background: #1877f2; color: white; border: none; border-radius: 3px; cursor: pointer;">
                    重新加载
                </button>
            </div>
        `;
    }
});



// 壁纸
App.use({
    name: 'BingWallpaperPlugin',
    
    async init() {
        await this.loadWallpaper();
    },
    
    async loadWallpaper() {
        const container = document.getElementById('bingWallpaper');
        if (!container) return;
        const proxyUrl = 'bing.php';

        try {
            container.innerHTML = '<div class="spinner"></div>';
            
            const response = await fetch(proxyUrl);
            if (!response.ok) {
                throw new Error(`网络请求失败: ${response.statusText}`);
            }
            const data = await response.json();
            if (data.error) {
                throw new Error(data.error);
            }

            const wallpaper = data.images[0];

            if (wallpaper) {
                const imageUrl = `https://cn.bing.com${wallpaper.url}`;
                const title = wallpaper.copyright;
                const link = wallpaper.copyrightlink;
                container.innerHTML = `
                    <a href="${imageUrl}" target="_blank" title="点击查看大图">
                        <img src="${imageUrl}" alt="${title}" loading="lazy">
                    </a>
                    <p class="wallpaper-title">
                        <a href="${link}" target="_blank" title="在必应中搜索相关信息">${title}</a>
                    </p>
                `;
            } else {
                throw new Error('未获取到壁纸数据');
            }
        } catch (error) {
            console.error('加载必应壁纸失败:', error);
            container.innerHTML = `<p style="text-align: center; color: #65676b; font-size: 0.875rem;">壁纸加载失败: ${error.message}</p>`;
        }
    }
});


// 友链
App.use({
    name: 'FriendLinksPlugin',
    
    async init() {
        await this.loadLinks();
    },
    
    async loadLinks() {
        const container = document.getElementById('linksList');
        if (!container) return;

        try {
            
            const response = await fetch('youlian.php');
            if (!response.ok) {
                throw new Error(`网络请求失败: ${response.statusText}`);
            }
            
            const data = await response.json();
            if (data.error) {
                throw new Error(data.error);
            }

            if (data && Array.isArray(data) && data.length > 0) {
                let html = '';
                data.forEach(link => {
                    if (link.name && link.url) {
                        html += `
                            <a href="${link.url}" target="_blank" rel="noopener noreferrer" class="link-tag">
                                ${link.name}
                            </a>
                        `;
                    }
                });
                container.innerHTML = html;
            } else {
                container.innerHTML = '<p style="text-align: center; color: #65676b; font-size: 0.875rem;">暂无链接</p>';
            }
        } catch (error) {
            console.error('加载友情链接失败:', error);
            container.innerHTML = `<p style="text-align: center; color: #ff4d4f; font-size: 0.875rem;">链接加载失败</p>`;
        }
    }
});




// 启动开始
document.addEventListener('DOMContentLoaded', () => App.init());




        // 移动菜单
        const menuToggle = document.getElementById('menuToggle');
        const headerNav = document.getElementById('headerNav');
        menuToggle.addEventListener('click', function() {
            headerNav.classList.toggle('active');
            menuToggle.classList.toggle('active');
        });
        
        document.addEventListener('click', function(event) {
            if (!headerNav.contains(event.target) && !menuToggle.contains(event.target)) {
                headerNav.classList.remove('active');
                menuToggle.classList.remove('active');
            }
        });
        
        window.addEventListener('resize', function() {
            if (window.innerWidth > 768) {
                headerNav.classList.remove('active');
                menuToggle.classList.remove('active');
            }
        });
        
const navData = {
    gonggao: { 
        title: '公告',
        paragraphs: [
            '本站友联征集中，要求：',
            '1.仅限导航网，或其他优质网站。',
            '2.长期发展，合法合规。',
            '3.内容丰富，保持更新。'
        ]
    },
    guanyu: { 
        title: '关于本站',
        paragraphs: [
            '本站（iare.cn）致力于提供优质的上网导航服务，和打造一流的聚合搜索引擎，以及重新定义后PC时代的导航站。我们的使命是让每个人都能轻松享受互联网的便利。',
            '简洁而多功能的界面设计，专注绝佳的用户体验。'
        ]
    },
    fankui: { 
        title: '意见反馈',
        paragraphs: [
            '有任何问题或建议，欢迎随时联系我们。',
            '邮箱：ule@outlook.com',
            'QQ：95231336'
        ]
    }
};

const navToastOverlay = document.getElementById('navToastOverlay');
const navToastTitle = document.getElementById('navToastTitle');
const navToastContent = document.getElementById('navToastContent');
const navToastCloseBtn = document.getElementById('navToastCloseBtn');
const headerNavLinks = document.querySelectorAll('.header-nav a');
function showNavToast(navType) {
    const data = navData[navType];
    if (!data) return;
    navToastTitle.textContent = data.title;
    navToastContent.innerHTML = data.paragraphs.map(para => 
        `<div class="nav-toast-paragraph">${para}</div>`
    ).join('');
    navToastOverlay.classList.add('active');
    clearTimeout(window.navToastTimer);
    window.navToastTimer = setTimeout(() => {
        hideNavToast();
    }, 30000);
}
function hideNavToast() {
    navToastOverlay.classList.remove('active');
}

headerNavLinks.forEach(link => {
    link.addEventListener('click', (e) => {
        e.preventDefault();
        const navType = link.getAttribute('href').substring(1);
        showNavToast(navType);
    });
});

navToastCloseBtn.addEventListener('click', hideNavToast);

navToastOverlay.addEventListener('click', (e) => {
    if (e.target === navToastOverlay) {
        hideNavToast();
    }
});

document.addEventListener('keydown', (e) => {
    if (e.key === 'Escape' && navToastOverlay.classList.contains('active')) {
        hideNavToast();
    }
});


// 禁用
var threshold = 200; 
var check = setInterval(function() {
    var widthDiff = window.outerWidth - window.innerWidth;
    var heightDiff = window.outerHeight - window.innerHeight;

    if (
        (widthDiff > threshold || heightDiff > threshold) &&
        !document.fullscreenElement
    ) {
        window.location.href = "about:blank";
    }
}, 1000);

document.addEventListener("keydown", function(e) {
    if (
        e.key === "F12" ||
        (e.ctrlKey && e.shiftKey && e.key === "I") ||
        (e.ctrlKey && e.shiftKey && e.key === "J") ||
        (e.ctrlKey && e.shiftKey && e.key === "C") ||
        (e.ctrlKey && e.key === "U")
    ) {
        e.preventDefault();
        window.location.href = "about:blank";
    }
});


setInterval(function() {
    var start = performance.now();
    debugger;
    var end = performance.now();
    if (end - start > 100) { 
        window.location.href = "about:blank";
    }
}, 1000);
