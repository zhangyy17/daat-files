(function($, document, window, undefined) {
    var view = getElementValue('#emergencyCall-viewData') || 'number';
    var DEFAULT_PAGEINDEX = 1,
        DEFAULT_PAGESIZE = 10,
        DEFAULT_TOTAL = 0,
        ADD_TIP = getElementValue('#emergencyCall-addAreaTxt'),
        UPDATE_TIP = getElementValue('#emergencyCall-editAreaTxt'),
        DELETE_TIP = getElementValue('#emergencyCall-deleteAreaTxt'),
        DELETE_AREA_CONFIRM = getElementValue('#emergencyCall-confirmDelAreaTxt'),
        SUCCESS_POPWIN_TITLE = getElementValue('#success'),

        ERROR_POPWIN_TITLE = getElementValue('#prompt'),
        COMFIRM_DELETE_TITLE = getElementValue('#prompt'),

        COMFIRM_SINGLE_DELETE = getElementValue('#emergencyCall-singleDeleteComfirm'),
        COMFIRM_BATCH_DELETE = getElementValue('#emergencyCall-batchDeleteConfirm'),
        COMFIRM_SCC_DELETE = getElementValue('#emergencyCall-sccDeleteConfirm'),
        COMFIRM_SCCS_DELETE = getElementValue('#emergencyCall-sccSDeleteConfirm');

    var NONE_SELECTOR_TIPS = {
        number: getElementValue('#emergencyCall-numberNoneSelector'),
        ip: getElementValue('#emergencyCall-ipNoneSelector'),
        mac: getElementValue('#emergencyCall-macNoneSelector'),
        scc: getElementValue('#emergencyCall-sccNoneSelector')
    };

    var CONFIRM_DELETE_TIPS = {
        number: getElementValue('#emergencyCall-numberDeleteConfirm'),
        ip: getElementValue('#emergencyCall-ipDeleteConfirm'),
        mac: getElementValue('#emergencyCall-macDeleteConfirm'),
        scc: getElementValue('#emergencyCall-sccDeleteConfirm')
    };

    var SELECT_CONVERTOR_FNS = {
        number: function(selectedItem) { return selectedItem.dn; },
        ip: function(selectedItem) { return selectedItem.nid; },
        mac: function(selectedItem) { return selectedItem.nid; }
    };

    var DELETE_FNS = {
        number: deleteNumber,
        ip: deleteIp,
        mac: deleteMac,
        scc: deleteSccNumber
    };
    //vue自定义拖拽指令
    Vue.directive('draggable', {
      bind: function (el) {
        el.style.position = 'absolute';
        var startX, startY, initialMouseX, initialMouseY;

        function mousemove(e) {
          var dx = e.clientX - initialMouseX;
          var dy = e.clientY - initialMouseY;
          if(startY+dy <70){
            dy =70 - startY
          }
          if(startX+dx<0){
            dx = -startX
          }
          if(startX + dx > getInner().width - el.offsetWidth){
            dx =   getInner().width-el.offsetWidth - startX
          }
          if(startY + dy > getInner().height - el.offsetHeight - 20){
            dy =   getInner().height-el.offsetHeight - startY -20
          }
          el.style.top = startY + dy + 'px';
          el.style.left = startX + dx + 'px';
          return false;
        }

        function mouseup() {
          document.removeEventListener('mousemove', mousemove);
          document.removeEventListener('mouseup', mouseup);
        }

        el.addEventListener('mousedown', function(e) {
          startX = el.offsetLeft;
          startY = el.offsetTop;
          initialMouseX = e.clientX;
          initialMouseY = e.clientY;
          document.addEventListener('mousemove', mousemove);
          document.addEventListener('mouseup', mouseup);
          return false;
        });
      }
    })
    var pageWidget = {};
    var searchor = {};
    var Modal = {
        // template: '<div class="modal" v-if="value"><slot></slot></div>',
        template: '<div class="modal" v-if="value"><div class="delete-result-popwin" v-bind:style="customizedStyle" v-draggable ><b class="pop-close" v-on:click="toggle"></b><h3 class="title"><slot name="title"></slot></h3><div class="table-box"><slot></slot></div><div class="bottom-box"><slot name="button-group"></slot></div></div></div>',
        props: {
            value: {
                type: Boolean,
                default: true,
            },
            width: {
                type: Number,
                default: 550
            },
            height: {
                type: Number,
                default: 350
            }
        },
        data: function() {
            return {
                style: { left: null, top: null, },
                state: { removable: false },
                drag: { x: null, y: null }
            };
        },
        computed: {
            customizedStyle: function() {
                return {
                    left: this.style.left + 'px',
                    top: this.style.top + 'px',
                    width: this.width + 'px',
                    height: this.height + 'px'
                };
            }
        },
        methods: {
            toggle: function() {
                this.$emit('input', !this.value);
            }
        },
        mounted: function() {
            var documentElement = document.documentElement,
                documentWidth = documentElement.clientWidth,
                documentHeight = documentElement.clientHeight,
                width = parseInt(this.width, 10),
                height = parseInt(this.height, 10),
                left = (documentWidth - width) / 2,
                top = (documentHeight - height) / 2;

            this.style.left = left;
            this.style.top = top;

        }
    };

    var pageVM = new Vue({
        el: '#emergencyCall-container',
        components: {
            'up-modal': Modal,
        },
        data: {
            selectedLocation: {
                id: null,
                name: null,
                upperLocationName: ''
            },
            currentView: view,
            selector: {
                number: [],
                ip: [],
                mac: [],
                scc: []
            },
            tableData: {
                number: [],
                ip: [],
                mac: [],
                scc: []
            },
            condition: {
                number: { searchKey: '', pushStatus: '0', defineLocation: '0' },
                mac: {
                    searchKey: '',
                    switchName: '',
                    location: '',
                    defineLocation: '0'
                },
                scc: {
                    searchKey: '',
                },
                ip: {
                    searchKey: '',
                    startIp: '',
                    endIp: '',
                    defineLocation: '0'
                }
            },
            filterState: {
                number: false,
                ip: false,
                mac: false,
                scc: false
            },
            state: {
                emptyPage: {
                    number: false,
                    ip: false,
                    mac: false,
                    scc: false
                },
                noneResult: {
                    number: false,
                    ip: false,
                    mac: false,
                    scc: false
                },
                isMove: false,

            },
            error: {
                visible: false,
                data: [],
                style: { left: '200px', top: '200px', },
                state: { removable: false },
                drag: { x: null, y: null }
            },
            importFile: {
                name: '',
                overSize: false
            },
            areaOperation: false
        },
        computed: {
            isSelectedAll: function() {
                var tableData = this.tableData[this.currentView],
                    selector = this.selector[this.currentView];
                if (!tableData) {
                    return false;
                }
                var tableDataLength = tableData.length;
                if (!tableDataLength) {
                    return false;
                }

                var slectorLength = selector.length;
                if (tableDataLength !== slectorLength) {
                    return false;
                }

                for (var idx = 0, len = tableDataLength; idx < len; ++idx) {
                    if (tableData[idx] !== selector[idx]) {
                        return false;
                    }
                }
                return tableDataLength === slectorLength;
            }
        },
        filters: {
            numberDetail: function(value) {
                if (!value) {
                    return '';
                }
                var param = {};
                var detail = {};
                detail.number = value.dn;
                detail.userName = value.userName;
                detail.pushStatus = value.pushStatus;
                detail.fullLocationName = value.fullLocationName;
                detail.location = { id: value.locationId, name: value.locationName };
                param.data = JSON.stringify(detail);
                return JSON.stringify(param);
            },
            macDetail: function(value) {
                if (!value) {
                    return '';
                }
                var param = {};
                var detail = {};
                detail.nid = value.nid;
                param.data = JSON.stringify(detail);
                return JSON.stringify(param);
            },
            ipDetail: function(value) {
                if (!value) {
                    return '';
                }
                var param = {};
                param.data = value.nid;
                return JSON.stringify(param);
            },
            sccDetail: function(value) {
                if (!value) {
                    return '';
                }
                var param = {};
                var detail = {};
                detail.sccNum = value.sccNum;
                detail.sccRealNum = value.sccRealNum;
                detail.defaultScc = value.defaultScc;
                detail.location = { id: value.locationId, name: value.locationName };
                detail.fullLocationName = value.fullLocationName;
                param.data = JSON.stringify(detail);
                return JSON.stringify(param);
            }
        },
        methods: {
            selectView: function(view) {
                this.currentView = view;
                if (view == 'scc') {
                    $("#emergencyCall-areaTree").removeClass('disabled');
                } else if (this.condition[view].defineLocation == '2') {
                    $("#emergencyCall-areaTree").addClass('disabled');
                }
                this.doSearch();
            },
            toggleFilterState: function(type) {
                this.filterState[type] = !this.filterState[type];
            },
            resetCondition: function() {
                var condition = this.condition[this.currentView];
                if (isString(condition)) {
                    this.condition[this.currentView] = '';
                } else if (isObject(condition)) {
                    var keys = Object.keys(condition);
                    for (var idx, len = keys.length; idx < len; ++idx) {
                        this.condition[this.currentView][keys[idx]] = '';
                    }
                } else {
                    throw 'Invalid Data Type';
                }
            },
            toggleAllSelector: function() {
                var numberSelector = this.selector[this.currentView],
                    numberData = this.tableData[this.currentView];
                if (numberSelector.length === numberData.length) {
                    while (numberSelector.length) {
                        numberSelector.pop();
                    }
                } else {
                    for (var idx = 0, len = numberData.length, number = null; idx < len; idx++) {
                        number = numberData[idx];
                        if (!numberSelector.contains(number)) {
                            numberSelector.push(number);
                        }
                    }
                }
            },
            toggleSelector: function(number) {
                var numberSelector = this.selector[this.currentView];
                if (numberSelector.contains(number)) {
                    numberSelector.splice(numberSelector.indexOf(number), 1);
                } else {
                    numberSelector.push(number);
                }
            },
            doSearch: function() {
                var currentView = this.currentView;
                var currentPageWidget = pageWidget[currentView];
                var condition = this.condition[currentView],
                    pageIndex = DEFAULT_PAGEINDEX,
                    pageSize = DEFAULT_PAGESIZE;
                if (currentPageWidget) {
                    pageIndex = currentPageWidget.getSelection();
                    pageSize = currentPageWidget.getPageSize();
                }
                if (currentView == 'ip' && !searchor[currentView]) {
                    searchor[currentView] = new UPORTAL.ErasableInput('#emergencyCall-ipEndSearchor', function() {
                        pageVM.condition[currentView].endIp = '';
                        pageVM.doSearch();
                    });
                    searchor[currentView] = new UPORTAL.ErasableInput('#emergencyCall-ipStartSearchor', function() {
                        pageVM.condition[currentView].startIp = '';
                        pageVM.doSearch();
                    });
                } else if (!searchor[currentView]) {
                    searchor[currentView] = new UPORTAL.ErasableInput('#emergencyCall-' + currentView + 'Searchor', function() {
                        pageVM.condition[currentView].searchKey = '';
                        pageVM.doSearch();
                    });
                }

                switch (currentView) {
                    case 'number':
                        queryNumber(this.selectedLocation.id, condition.searchKey, pageIndex, pageSize, condition.pushStatus, condition.defineLocation);
                        break;
                    case 'scc':
                        querySccNumber(this.selectedLocation.id, condition.searchKey, pageIndex, pageSize);
                        break;
                    case 'mac':
                        queryMac(this.selectedLocation.id, condition.searchKey, pageIndex, pageSize, condition.defineLocation);
                        break;
                    case 'ip':
                        queryIp(this.selectedLocation.id, condition.startIp, condition.endIp, pageIndex, pageSize, condition.defineLocation);
                        break;
                    default:
                        // TODO Something
                }
            },
            doDelete: function() {
                var view = this.currentView;
                var selector = this.selector[view];
                if (!selector.length) {
                    showErrorPop(ERROR_POPWIN_TITLE, NONE_SELECTOR_TIPS[view]);
                } else {
                    showConfirmPop(COMFIRM_DELETE_TITLE, CONFIRM_DELETE_TIPS[view], function() {
                        var fn = SELECT_CONVERTOR_FNS[view];
                        if (fn) {
                            selector = selector.map(fn);
                        }
                        DELETE_FNS[view](selector);
                    });
                }
            },
            toggleSwitch: function() {
                this.state.isMove = !this.state.isMove;
            },
            toggleErrorPopwin: function() {
                this.error.visible = !this.error.visible;
            },
            downloadTemplate: function(view) {
                switch (view) {
                    case 'number':
                        window.location.href = "downUserLocationImpTemplate.sraction";
                        break;
                    case 'mac':
                        window.location.href = "downMacLocationImpTemplate.sraction";
                        break;
                    case 'scc':
                        window.location.href = "downSCCNumTemplate.sraction";
                        break;
                    case 'ip':
                        window.location.href = "downIpLocationImpTemplate.sraction";
                        break;
                    case 'area':
                        window.location.href = "downLocationDefTemplate.sraction";
                        break;
                    default:
                        // TODO Something
                }
            },
            clearImportFile: function() {
                this.importFile.name = '';
                this.importFile.overSize = false;
                $("#emergencyCall-fileInputId").val("");
                $("#emergencyCall-FormatErrorAlert").hide();
            },
            getImportFileName: function(event) {
                var el = event.currentTarget;
                var filePath = el.value;
                var pos2 = filePath.lastIndexOf('\\');
                this.importFile.name = (filePath.substring(pos2 + 1));
                if (this.importFile.name != "") {
                    $("#emergencyCall-FormatErrorAlert").hide();
                }
            },
            upLoadFile: function(view) {
                if (!checkFileName()) {
                    return;
                }
                if (!checkFileSize()) {
                    return;
                }
                var url = '';
                switch (view) {
                    case 'number':
                        url = "importUserLocation.sraction";
                        //任务管理跳转到相应的页签
                        $("#main-siteTaskType").val(12);
                        break;
                    case 'mac':
                        url = "importMacLocation.sraction";
                        $("#main-siteTaskType").val(14);
                        break;
                    case 'scc':
                        url = "uploadSCCNumExcel.sraction";
                        $("#main-siteTaskType").val(15);
                        break;
                    case 'ip':
                        url = "importIpLocation.sraction";
                        $("#main-siteTaskType").val(13);
                        break;
                    case 'area':
                        url = "uploadLocationDefExcel.sraction";
                        $("#main-siteTaskType").val(11);
                        //url="uploadMemberExcel.sraction";
                        break;
                    default:
                }
                importExcelFile(url);
            },
            exportFile: function(view) {
                var data;
                switch (view) {
                    case 'number':
                        data = {
                            "userLocationListVO.locationId": this.selectedLocation.id,
                            "userLocationListVO.dn": this.condition[view].searchKey,
                            "userLocationListVO.pushStatus": this.condition[view].pushStatus,
                            "userLocationListVO.defineLocation": this.condition[view].defineLocation,
                        };
                        exportExcelFile("exportUserLocation.sraction", data);
                        $("#main-siteTaskType").val(12);
                        break;
                    case 'mac':
                        data = {
                            "searchVO.locationId": this.selectedLocation.id,
                            "searchVO.searchKey": this.condition[view].searchKey,
                            "searchVO.locationType": 0,
                            "searchVO.defineLocation": this.condition[view].defineLocation,
                        };
                        exportExcelFile("exportMacLocation.sraction", data);
                        $("#main-siteTaskType").val(14);
                        break;
                    case 'scc':
                        data = {
                            "exportSCCNumVo.locationId": this.selectedLocation.id,
                            "exportSCCNumVo.searchKey": this.condition[view].searchKey,
                        };
                        exportExcelFile("exportSCCNumFile.sraction", data);
                        $("#main-siteTaskType").val(15);
                        break;
                    case 'ip':
                        data = {
                            "searchVO.locationType": 1,
                            "searchVO.locationId": this.selectedLocation.id,
                            "searchVO.startIp": this.condition[view].startIp,
                            "searchVO.endIp": this.condition[view].endIp,
                            "searchVO.defineLocation": this.condition[view].defineLocation
                        };
                        exportExcelFile("exportIpLocation.sraction", data);
                        $("#main-siteTaskType").val(13);
                        break;
                    case 'area':
                        exportExcelFile("exportLocationDefFile.sraction");
                        $("#main-siteTaskType").val(11);
                        break;
                    default:
                }
            },
            setAreaOperation: function() {
                this.areaOperation = true;
            },
            resetAreaOperation: function() {
                this.areaOperation = false;
            }
        }
    }); //end of pageVM

    //虚线拖动
    drag($("#emergencyCall-switchLine"), $("#emergencyCall-switchPopwin"), $("#emergencyCall-rightBox"), $("#emergencyCall-lineText"));
    initPopups();
    var treeWidget = null;

    buildTreeWidth();
    $('#emergencyCall-popImport').off('change', '#emergencyCall-fileInputId').on('change', '#emergencyCall-fileInputId', function(event) {
        $parentImportBox = $(this).closest(".pop-import-box");
        var filePath = $parentImportBox.find(".importfile").val();
        var pos2 = filePath.lastIndexOf('\\');
        $parentImportBox.find(".file-name-input").val(filePath.substring(pos2 + 1));
        pageVM.importFile.name = filePath.substring(pos2 + 1);
        $("#emergencyCall-fileSizeAlert").hide();
        if (filePath.substring(pos2 + 1) != "") {
            $("#emergencyCall-FormatErrorAlert").hide();
        }
    });
    $("#emergencyCall-content").off("click.droplist").on("click.droplist", function() {
        $(".ec-droplist .ec-droplist-select").removeClass('active').siblings().hide();
    });
    $("#emergencyCall-rightBox").off("click.droplist", ".ec-droplist .ec-droplist-select").on("click.droplist", ".ec-droplist .ec-droplist-select", function(e) {
        e.stopPropagation();
        $(".ec-droplist .ec-droplist-select").not($(this)).removeClass('active').siblings('.ec-droplist-option').slideUp(200); //打开一个下拉框时，其他展开的下拉框收起来
        $(this).toggleClass("active").siblings('.ec-droplist-option').slideToggle(200);
    });
    $("#emergencyCall-pushSelect").off("click", ".ec-droplist-option > li").on("click", ".ec-droplist-option > li", function() {
        var txt = $(this).text();
        var status = $(this).attr("data-pushStatus");
        $(this).closest('.ec-droplist').find(".ec-droplist-select").text(txt);
        $(this).closest('.ec-droplist').find(".ec-droplist-select").attr("data-pushStatus", status);
        $(this).closest('.ec-droplist').find(".ec-droplist-select").attr("title", txt);
        pageVM.condition.number.pushStatus = status;
        pageVM.doSearch();
    });
    //defineLocation 0-全部 1 -已定义 2-未定义
    $(".define-location-droplist").off("click", ".ec-droplist-option > li").on("click", ".ec-droplist-option > li", function() {
        $areaTree = $("#emergencyCall-areaTree");
        var txt = $(this).text();
        var defineLocation = $(this).attr("data-defineLocation");
        if (defineLocation == "0" || defineLocation == "1") {
            $areaTree.removeClass('disabled');
        } else if (defineLocation == "2") {
            $areaTree.addClass('disabled');
        }
        $(this).closest('.ec-droplist').find(".ec-droplist-select").text(txt);
        $(this).closest('.ec-droplist').find(".ec-droplist-select").attr("data-defineLocation", defineLocation);
        $(this).closest('.ec-droplist').find(".ec-droplist-select").attr("title", txt);
        pageVM.condition[pageVM.currentView].defineLocation = defineLocation;
        if (defineLocation == "2" && pageVM.selectedLocation.id != "1") {
            $('.ucd-tree-node[rid="1"] > .ucd-tree-header a').trigger('click');
        }
        pageVM.doSearch();
    });

    function buildTreeWidth() {
        requestJson({
            url: "queryMeetingRoomArea.sraction",
            data: function() {
                return { code: 1, isLazy: true };
            },
            success: function(response) {
                if (!response) {
                    return;
                }

                if (treeWidget) {
                    $('#emergencyCall-areaTree').empty();
                    treeWidget.destroy();
                }

                treeWidget = _buildTreeWidget(response.data);
                treeWidget.toggleNode("1", true);

                function _buildTreeWidget(treeData) {
                    return new UCD.Tree('#emergencyCall-areaTree', {
                        idField: 'code',
                        hasChildNode: 'hasChild',
                        childrenField: 'children',
                        dragdrop: false,
                        data: [treeData],
                        titleField: 'name',
                        onCreateNode: function($node, detail) {
                            var $nodeContent = $node.find('a');
                            if (detail.depth < 6) {
                                $nodeContent.append(_buildOperateIcon('add', ADD_TIP));
                                $nodeContent.find(".add").attr("title", ADD_TIP);
                            }
                            $nodeContent.append(_buildOperateIcon('update', UPDATE_TIP));
                            $nodeContent.find(".update").attr("title", UPDATE_TIP);
                            // 只有叶子节点可以删除
                            if (detail.depth !== 0) {
                                $nodeContent.append(_buildOperateIcon('delete', DELETE_TIP));
                                $nodeContent.find(".delete").attr("title", DELETE_TIP);
                            }

                            // 隐藏非根节点的Icon
                            if (detail.parentCode) {
                                $nodeContent.find('.icon').hide();
                            }

                            function _buildOperateIcon(type, tip) {
                                return '<p class="icon ' + type + '"></p>';
                            }
                        },
                        onToggleNode: function(e, $node) {
                            var code = getElementAttribute($node, 'rid');
                            if (parseInt($node.find(".ucd-tree-header").find(".ucd-tree-ocl").attr("level")) > 1) {
                                $(".line-tree").addClass("scroll-class");
                            }
                            // 只有展开时，才需要更新数据
                            if (!$node.hasClass('open')) {
                                updateTreeNode(code);
                            } else {
                                $node.find('.ucd-tree-content').empty();
                            }
                        },
                        create: function() {
                            var $tree = $(this);
                            $tree
                                .on('click', '.add', function(event) {
                                    event = event || window.event;
                                    event.stopPropagation();

                                    var $this = $(this),
                                        districtName = getElementAttribute($this.parent(), 'title'),
                                        districtCode = getElementAttribute($this.closest('[rid]'), 'rid');
                                    requestHtml({
                                        data: {
                                            data: JSON.stringify({ isEmergency: true })
                                        },
                                        url: 'toAddMeetingRoomArea.sraction',
                                        targetSelector: '.frame-right-side',
                                        success: function(html) {
                                            $("#addMeetingRoomDistrict-upperDistrict ").val(districtName);
                                            $("#addMeetingRoomDistrict-upperDistrictCode").val(districtCode);
                                        },
                                    });
                                })
                                .on('click', '.update', function(event) {
                                    event = event || window.event;
                                    event.stopPropagation();

                                    var districtCode = getElementAttribute($(this).closest('[rid]'), 'rid');
                                    requestHtml({
                                        data: { code: districtCode, from: 'emergency' },
                                        url: 'toEditMeetingRoomArea.sraction',
                                        targetSelector: '.frame-right-side'
                                    });
                                })
                                .on('click', '.delete', function(event) {
                                    event = event || window.event;
                                    event.stopPropagation();
                                    var $this = $(this),
                                        $currentNode = $this.closest('[rid]'),
                                        $parentNode = $currentNode.parent().closest('[rid]'),
                                        code = getElementAttribute($currentNode, 'rid'),
                                        parentCode = getElementAttribute($parentNode, 'rid');
                                    showConfirmPop(COMFIRM_DELETE_TITLE, DELETE_AREA_CONFIRM, function() {
                                        requestJson({
                                            data: function() {
                                                var data = {
                                                    code: code
                                                };
                                                return data;
                                            },
                                            url: "delMeetingRoomArea.sraction",
                                            success: function(response) {
                                                if (parentCode === "1") {
                                                    buildTreeWidth();
                                                } else {
                                                    updateTreeNode(parentCode);
                                                    $parentNode.find('> .ucd-tree-header a').click();
                                                }
                                            }
                                        });
                                    });
                                })
                                .on('click', '.ucd-tree-header a', function() {
                                    var tmpId = getElementAttribute($(this).closest('.ucd-tree-node'), 'rid');
                                    $TreeContainer = $("#emergencyCall-areaTree");
                                    if (($TreeContainer.hasClass('disabled') && tmpId != '1') || ($TreeContainer.hasClass('firstnode-disabled') && tmpId == '1')) {
                                        return;
                                    }
                                    $tree.find('.ucd-tree-header a').not(this).removeClass('selected');
                                    $(this).addClass('selected');

                                    pageVM.selectedLocation.id = tmpId;
                                    pageVM.selectedLocation.name = getElementAttribute(this, 'title');
                                    if ($(this).closest('.ucd-tree-node').find("span.ucd-tree-ocl").attr("level") === "0") {
                                        pageVM.selectedLocation.upperLocationName = '';
                                    } else {
                                        pageVM.selectedLocation.upperLocationName = getElementAttribute($(this).closest('.ucd-tree-node').parent().parent().find(".ucd-tree-header").find("a"), 'title');
                                    }

                                    pageVM.doSearch();
                                })
                                .on('mouseenter', '.ucd-tree-header a', function() {
                                    var $this = $(this),
                                        $node = $this.closest('.ucd-tree-node'),
                                        $icons = $this.find('.icon');
                                    $node.addClass('selected');
                                    if ($node.hasClass('ucd-tree-leaf')) {
                                        $icons.show();
                                    } else {
                                        $icons.not('.delete').show();
                                    }
                                })
                                .on('mouseleave', '.ucd-tree-header a', function() {
                                    var $this = $(this),
                                        $node = $this.closest('.ucd-tree-node'),
                                        $icons = $this.find('.icon'),
                                        code = getElementAttribute($node, 'rid');

                                    $node.removeClass('selected');
                                    if (code !== "1") {
                                        $icons.hide();
                                    }
                                });
                        }
                    });
                }
            }
        }); // end of requestJson
    }


    // 选中根节点
    $('.ucd-tree-node[rid="1"] > .ucd-tree-header a').trigger('click');

    /**
     * 查询用户号码
     * @param  {String}   locationId 区域ID
     * @param  {String}   number     用户号码
     * @param  {Number}   pageIndex  当前页码
     * @param  {Number}   pageSize   每页记录数
     * @param  {Number}   pushStatus   上报状态
     * @param  {Number}   defineLocation   位置定义
     * @return {Function}            Function
     */
    function queryNumber(locationId, number, pageIndex, pageSize, pushStatus, defineLocation) {
        requestJson({
            data: function() {
                return {
                    'userLocationListVO.locationId': defineLocation == "2" ? "1" : locationId,
                    'userLocationListVO.dn': number,
                    'userLocationListVO.pageIndex': pageIndex || DEFAULT_PAGEINDEX,
                    'userLocationListVO.pageSize': pageSize || DEFAULT_PAGESIZE,
                    'userLocationListVO.pushStatus': pushStatus,
                    'userLocationListVO.defineLocation': defineLocation
                };
            },
            url: 'queryUserLocationList_json.sraction',
            success: function(response) {
                querySuccessCallback('number', response, queryNumber);
            },
            fail: function(response) {
                queryFailureCallback('number', response, queryNumber);
            }
        }); //end of requestJson
    }

    /**
     * 删除用户号码
     * @param  {Array}    numbers 待删除的号码列表
     * @return {Function}         Function
     */
    function deleteNumber(numbers) {
        requestJson({
            data: function() {
                var data = {};
                for (var idx = 0, len = numbers.length; idx < len; ++idx) {
                    data['userLocationDelVO.userLocationBeanList[' + idx + '].dn'] = numbers[idx];
                }
                return data;
            },
            url: 'delUserLocation.sraction',
            success: deleteSuccessCallback,
            fail: deleteFailureCallback
        });
    } // end of deleteNumber

    function queryMac(locationId, searchKey, pageIndex, pageSize, defineLocation) {
        requestJson({
            validate: function() {
                return true;
            },
            data: function() {
                return {
                    'searchVo.locationType': 0,
                    'searchVo.locationId': defineLocation == "2" ? "1" : locationId,
                    'searchVo.pageIndex': pageIndex || DEFAULT_PAGEINDEX,
                    'searchVo.pageSize': pageSize || DEFAULT_PAGESIZE,
                    'searchVo.searchKey': searchKey,
                    'searchVo.defineLocation': defineLocation,
                };
            },
            url: 'physicallocation_search.sraction',
            success: function(response) {
                querySuccessCallback('mac', response, queryMac);
            },
            fail: function(response) {
                queryFailureCallback('mac', response, queryMac);
            }
        }); //end of requestJson
    }

    function deleteMac(ids) {
        requestJson({
            data: function() {
                var data = {};
                for (var idx = 0, len = ids.length; idx < len; ++idx) {
                    data['delVo[' + idx + ']'] = ids[idx];
                }
                return data;
            },
            url: 'physicallocation_del.sraction',
            success: deleteSuccessCallback,
            fail: deleteFailureCallback
        });
    }

    function queryIp(locationId, startIp, endIp, pageIndex, pageSize, defineLocation) {
        requestJson({
            validate: function() {
                return true;
            },
            data: function() {
                return {
                    'searchVo.locationType': 1,
                    'searchVo.locationId': defineLocation == "2" ? "1" : locationId,
                    'searchVo.pageIndex': pageIndex || DEFAULT_PAGEINDEX,
                    'searchVo.pageSize': pageSize || DEFAULT_PAGESIZE,
                    'searchVo.startIp': startIp,
                    'searchVo.endIp': endIp,
                    'searchVo.defineLocation': defineLocation
                };
            },
            url: 'physicallocation_search.sraction',
            success: function(response) {
                querySuccessCallback('ip', response, queryIp);
            },
            fail: function(response) {
                queryFailureCallback('ip', response, queryIp);
            }
        }); //end of requestJson
    }

    function deleteIp(ids) {
        requestJson({
            data: function() {
                var data = {};
                for (var idx = 0, len = ids.length; idx < len; ++idx) {
                    data['delVo[' + idx + ']'] = ids[idx];
                }
                return data;
            },
            url: 'physicallocation_del.sraction',
            success: deleteSuccessCallback,
            fail: deleteFailureCallback
        });
    }

    /**
     * 查询SCC路由号码
     * @param  {Number}   pageIndex 当前页码
     * @param  {Number}   pageSize  每页记录数
     * @return {Function}           Function
     */
    function querySccNumber(locationId, searchKey, pageIndex, pageSize) {
        requestJson({
            validate: function() {
                return true;
            },
            data: function() {
                return {
                    'vo.locationId': locationId,
                    'vo.pageIndex': pageIndex || DEFAULT_PAGEINDEX,
                    'vo.pageSize': pageSize || DEFAULT_PAGESIZE,
                    'vo.searchKey': searchKey,
                };
            },
            url: 'querySCCNum.sraction',
            success: function(response) {
                querySuccessCallback('scc', response, querySccNumber);
            },
            fail: function(response) {
                queryFailureCallback('scc', response, querySccNumber);
            }
        }); //end of requestJson

    } //end of querySccNumber

    /**
     * 删除SCC号码
     * @param  {Array}    numbers 待删除的SCC信息
     * @return {Function}         Function
     */
    function deleteSccNumber(numbers) {
        requestJson({
            data: function() {
                var data = {};
                for (var idx = 0, len = numbers.length; idx < len; ++idx) {
                    data['vo.sccNumBeanList[' + idx + '].locationId'] = numbers[idx].locationId;
                    data['vo.sccNumBeanList[' + idx + '].sccNum'] = numbers[idx].sccNum;
                    data['vo.sccNumBeanList[' + idx + '].sccRealNum'] = numbers[idx].sccRealNum;
                }
                return data;
            },
            url: 'delSCCNum.sraction',
            success: deleteSuccessCallback,
            fail: deleteFailureCallback
        });
    } // end of deleteNumber

    /**
     * 创建分页组件
     * @param  {String|jQuery Selector} container 控件选择器
     * @param  {Function}               fn        分页信息改变时所触发的回调函数
     * @return {UCD.Pages}                        分页组件
     */
    function createPagingWidget(container, fn) {
        var pageWidget = new UCD.Pages(container, DEFAULT_TOTAL, DEFAULT_PAGESIZE, [DEFAULT_PAGESIZE, 2 * DEFAULT_PAGESIZE, 5 * DEFAULT_PAGESIZE, 10 * DEFAULT_PAGESIZE], DEFAULT_PAGEINDEX, function() {
            // 只有pageWidget完成初始化时，才允许执行以下操作。
            // 原因：Page组件在初始化的时候也会触发此回调，但不应该触发。
            if (pageWidget && fn) {
                fn(pageWidget.getSelection(), pageWidget.getPageSize());
            }
        });
        ucdWidgets.push(pageWidget);
        return pageWidget;
    }

    /**
     * 查询成功的回调函数
     * @param  {String}   view     查询实体
     * @param  {Object}   response 查询结果
     * @return {Function}          Function
     */
    function querySuccessCallback(view, response, fn) {
        var responseData = response.data;
        pageVM.tableData[view] = responseData.data || [];
        if (view == 'ip') {
            pageVM.state.emptyPage[view] = !responseData.totalCount && !pageVM.condition[view].startIp && !pageVM.condition[view].endIp;
            pageVM.state.noneResult[view] = !responseData.totalCount && (pageVM.condition[view].startIp || pageVM.condition[view].endIp);
        } else {
            pageVM.state.emptyPage[view] = !responseData.totalCount && !pageVM.condition[view].searchKey;
            pageVM.state.noneResult[view] = !responseData.totalCount && pageVM.condition[view].searchKey;
        }
        if (!pageWidget[view]) {
            pageWidget[view] = createPagingWidget('#emergencyCall-' + view + 'Page', function(pageIndex, pageSize) {
                if (view == 'number') {
                    fn(pageVM.selectedLocation.id, pageVM.condition[view].searchKey, pageIndex, pageSize, pageVM.condition[view].pushStatus, pageVM.condition[view].defineLocation);
                } else if (view == 'ip') {
                    fn(pageVM.selectedLocation.id, pageVM.condition[view].startIp, pageVM.condition[view].endIp, pageIndex, pageSize, pageVM.condition[view].defineLocation);
                } else if (view == 'mac') {
                    fn(pageVM.selectedLocation.id, pageVM.condition[view].searchKey, pageIndex, pageSize, pageVM.condition[view].defineLocation);
                } else {
                    fn(pageVM.selectedLocation.id, pageVM.condition[view].searchKey, pageIndex, pageSize);
                }
            });
        }
        var chekboxSelector = pageVM.selector[view];
        while (chekboxSelector.length) {
            chekboxSelector.pop();
        }
        pageWidget[view].setTotal(responseData.totalCount);
        pageWidget[view].setSelection(responseData.pageIndex);
    }

    /**
     * 查询失败的回调函数
     * @param  {String}   view     查询实体
     * @param  {Object}   response 查询结果
     * @return {Function}          Function
     */
    function queryFailureCallback(view, response) {
        // showErrorPop(ERROR_POPWIN_TITLE, response.text, function() {
        pageVM.tableData[view] = [];
        pageVM.state.emptyPage[view] = false;
        pageVM.state.noneResult[view] = false;
        // });
        var chekboxSelector = pageVM.selector[view];
        while (chekboxSelector.length) {
            chekboxSelector.pop();
        }
    }

    function deleteSuccessCallback(response) {
        showSuccessPop(SUCCESS_POPWIN_TITLE, function() {
            pageVM.doSearch();
        });
        pageVM.selector[pageVM.currentView] = [];
    }

    function deleteFailureCallback(response) {
        pageVM.error.data = [];
        var idx,
            item,
            responseData = response.data;
            //适配只读权限批量失败
            if(response.data == null){
                showErrorPop($("#error").val(), response.text);
            }else{
                        pageVM.error.visible = true;
        switch (pageVM.currentView) {
            case 'number':
                responseData = responseData.resultList;
                responseDataLength = responseData.length;
                for (idx = 0; idx < responseDataLength; ++idx) {
                    item = responseData[idx];
                    pageVM.error.data.push({ key: item.userLocationBean.dn, value: item.errorDesc });
                }
                break;
            case 'mac':
                responseDataLength = responseData.length;
                if (responseData && responseDataLength) {
                    for (idx = 0; idx < responseDataLength; ++idx) {
                        item = responseData[idx];
                        pageVM.error.data.push({ key: item.locationItem, value: item.details });
                    }
                }
                break;
            case 'scc':
                responseData = responseData.resultList;
                responseDataLength = responseData.length;
                for (idx = 0; idx < responseDataLength; ++idx) {
                    item = responseData[idx];
                    pageVM.error.data.push({ key: item.sccNumBean.sccNum, value: item.errorDesc });
                }
                break;
            case 'ip':
                responseDataLength = responseData.length;
                if (responseData && responseDataLength) {
                    for (idx = 0; idx < responseDataLength; ++idx) {
                        item = responseData[idx];
                        pageVM.error.data.push({ key: item.locationItem, value: item.details });
                    }
                }
                break;
            default:

        }
            }

        pageVM.selector[pageVM.currentView] = [];
    }

    function updateTreeNode(code) {
        requestJson({
            url: "queryMeetingRoomArea.sraction",
            async: true,
            loding: true,
            data: {
                "code": code,
                "isLazy": true
            },
            success: function(response) {
                var node = response.data;
                if (code !== '1') {
                    treeWidget.setNode(code, node);
                }
                var children = node.children;
                if (children && children.length) {
                    treeWidget.addNode(code, children);
                }
                treeWidget.toggleNode(code, true);
            }
        });
    }

    /**
     * 虚线拖动
     */
    function drag(_move, _popwin, _rightBox, _bottomBox) {
        var $document = $(document);
        _move.on("mousedown", function(e) {
            $(_popwin).css({
                transition: 'initial'
            });

            e.preventDefault();
            e.stopPropagation();
            moveable = true;
            var diffX = e.clientX,
                width = _popwin.width(),
                width2 = parseInt(_rightBox
                    .css("margin-left"));
            width3 = _bottomBox.width();
            width4 = $(".user").width();
            if (typeof _popwin.setCapture != 'undefined') {
                _popwin.setCapture();
            }
            $document.on("mousemove", mousemove);

            function mousemove(e) {
                e.preventDefault();
                e.stopPropagation();
                if (!moveable) {
                    return;
                }
                var left = e.clientX - diffX;
                var changeWidth = width + left;
                var changeWidth2 = width2 + left;
                var changeWidth3 = width3 + left;
                var changeWidth4 = width4 - left;
                if (changeWidth3 < 360 || changeWidth4 < 820) {
                    return false;
                }
                if (changeWidth > 0) {
                    _popwin.width(changeWidth);
                    _rightBox.css("margin-left", changeWidth2);
                    _bottomBox.width(changeWidth3);
                }

            }
            $document.on("mouseup", mouseup);

            function mouseup(e) {
                e.preventDefault();
                e.stopPropagation();
                moveable = null;
                $document.off("mousemove", mousemove);
                $document.off("mouseup", mouseup);
                if (typeof _popwin.releaseCapture != 'undefined') {
                    _popwin.releaseCapture();
                }
            }
        });
    }
    /**
     *  检查上传的文件是否为xls、xlsx
     * @return {[Boolean]} true xls、xlsx格式正确
     */
    function checkFileName() {
        var fileName = pageVM.importFile.name;
        var suffix = fileName.substr(fileName.lastIndexOf('.') + 1);
        if (suffix != "xlsx" && suffix != "xls") {
            $("#emergencyCall-FormatErrorAlert").show();
            return false;
        }
        $("#emergencyCall-FormatErrorAlert").hide();
        return true;
    }
    /**
     *  检查上传的文件大小
     * @return {[Boolean]} true  文件小于4M
     */
    function checkFileSize() {
        var flag = true;
        var files = document.getElementById("emergencyCall-fileInputId").files;
        if (files && files.length) {
            file = files[0];
            size = file.size;
            if (4194304 < size) {
                pageVM.importFile.overSize = true;
                flag = false;
                return flag;
            }
        }
        pageVM.importFile.overSize = false;
        return flag;
    }
    /**
     * 上传文件
     * @param  类型 view (用户号码、IP号段、MAC地址、SCC路由号码)
     */
    function importExcelFile(url) {
        var isIE = "0";
        var file = null,
            size = 0;
        if (parseInt($.browser.version) <= 11) {
            isIE = "1";
        }
        $.ajaxFileUpload({
            url: url,
            secureuri: true,
            fileElementId: 'emergencyCall-fileInputId',
            dataType: "json",
            data: {
                'ie': 1
            },
            success: function(msg) {
                if (msg.text == "success") {
                    $("#emergencyCall-popImport,#mask_layer").hide();
                    $(".frame-right-side").removeClass("scrollNone");
                    showSuccessPop($("#success").val(), function() {
                        $("#menu_corpLBS").trigger("click");
                        taskResult();
                    });
                } else {
                    var filePath = $("#emergencyCall-fileInputId").val();
                    var pos2 = filePath.lastIndexOf('\\');
                    if (filePath.substring(pos2 + 1) == '') {
                        pageVM.importFile.name = "";
                    }
                    $("#emergencyCall-importError").text(msg.text)
                    return false;
                }
            }
        });
    }
    /**
     *  exportExcelFile 导出excel
     * @param  {[String]} url  导出的action
     * @param  {[Onject]} data 传的数据
     */
    function exportExcelFile(url, data) {
        requestJson({
            type: "POST",
            url: url,
            async: false,
            data: data,
            dataType: "json",
            successResults: ["Succeeded", "success"],
            success: function(msg) {
                if (msg.code == 0 || msg.text == 'success') {
                    showSuccessPop($("#success").val(), function() {
                        taskResult(); // 分发任务查询,弹出框view事件
                    });
                } else {
                    showErrorPop($("#error").val(), msg.text);
                }
            }
        });
    }

    /**
     * 分发任务弹出窗
     */
    function taskResult() {
        $("#memTask-msgCard").css("display", "block");
        $("#memTask-msgCard-btn").css("opacity", "0");
        $("#memTask-msgCard").animate({
            height: '160px',
            width: '380px'
        }, 500, function() {
            $("#memTask-msgCard-btn").animate({
                opacity: '1',
            }, 'fast');
        });

        setTimeout(function() {
            $("#memTask-msgCard-btn").animate({
                opacity: '0',
            }, 'fast', function() {
                $("#memTask-msgCard").animate({
                    height: '0px',
                    width: '0px'
                });
                $("#memTask-msgCard-btn").css("opacity", "1");
            });
        }, 5000);
    }

}(jQuery, document, window));
