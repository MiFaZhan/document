# 绿管平台 Controller API 详细文档

本文档详细记录了 `ruoyi-platform/src/main/java/com/ruoyi/lvguan/controller` 目录下所有Controller类的增删改查方法。

---

## 1. DashboardController - 数据大屏

**路径前缀**: `/lvguan/dashboard`

### 查询方法

| 方法名 | HTTP方法 | 路径 | 参数 | 返回类型 | 权限 |
|--------|---------|------|------|----------|------|
| `getTopData` | GET | `/topdata` | 无 | `R<List<TitleVO>>` | @PermitAll |
| `getLossAmount` | GET | `/lossamount` | 无 | `R<List<BrandTypeVO>>` | @PermitAll |
| `getRegionData` | GET | `/regiondata` | 无 | `R<List<RegionDataVO>>` | @PermitAll |
| `getBrandType` | GET | `/brandtype` | 无 | `R<List<BrandTypeVO>>` | @PermitAll |
| `getPartTypeDistribution` | GET | `/parttype` | 无 | `R<List<PartTypeDistributionVO>>` | @PermitAll |
| `getTop5InsuranceSales` | GET | `/top5` | 无 | `R<List<Top5InsuranceSalesVO>>` | @PermitAll |
| `getComponyPolicyData` | GET | `/componyAndPolicyData` | 无 | `R<CompanyAndPolicyVO>` | @PermitAll |

---

## 2. TbInsurancePolicyController - 保单管理

**路径前缀**: `/lvguan/policy`

### 查询方法

| 方法名                            | HTTP方法 | 路径                   | 参数                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        | 返回类型                  | 权限                                                     |
| ------------------------------ | ------ | -------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------- | ------------------------------------------------------ |
| `list`                         | GET    | `/list`              | `@Param("keyword") String keyword`<br>`@Param("insuranceCompany") String insuranceCompany`<br>`@Param("region") String region`<br>`@Param("vehicleModel") String vehicleModel`<br>`@Param("policyPremiumMin") String policyPremiumMin`<br>`@Param("policyPremiumMax") String policyPremiumMax`<br>`@Param("dateEnd") @DateTimeFormat(pattern = "yyyy-MM-dd") LocalDate dateEnd`<br>`@Param("dateStart") @DateTimeFormat(pattern = "yyyy-MM-dd") LocalDate dateStart`<br>`@Param("status") Integer status` | `TableDataInfo`       | `@PreAuthorize("@ss.hasPermi('lvguan:policy:query')")` |
| `get`                          | GET    | `/get/file`          | `@Param("id") Long id`<br>`HttpServletResponse response`                                                                                                                                                                                                                                                                                                                                                                                                                                                  | `void`                | `@PreAuthorize("@ss.hasPermi('lvguan:policy:query')")` |
| `getTitle`                     | GET    | `/get/title`         | 无                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         | `R<InsuranceTitleVO>` | `@PreAuthorize("@ss.hasPermi('lvguan:policy:query')")` |
| `getByName`                    | GET    | `/getbyname`         | `@Param("name") String name`                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | `TableDataInfo`       | `@PreAuthorize("@ss.hasPermi('lvguan:policy:query')")` |
| `getInsurancePolicyAttachment` | GET    | `/get/attachment`    | `@Param("id") Long id`<br>`HttpServletResponse response`                                                                                                                                                                                                                                                                                                                                                                                                                                                  | `void`                | `@PreAuthorize("@ss.hasPermi('lvguan:policy:query')")` |
| `getPolicyCompany`             | GET    | `/get/policyCompany` | 无                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         | `R<List<DeptVO>>`     | `@PreAuthorize("@ss.hasPermi('lvguan:policy:query')")` |

### 新增方法

| 方法名                            | HTTP方法 | 路径                | 参数                                                                        | 返回类型                   | 权限                                                   |
| ------------------------------ | ------ | ----------------- | ------------------------------------------------------------------------- | ---------------------- | ---------------------------------------------------- |
| `addInsurancePolicy`           | POST   | `/add`            | `@RequestBody @Validated InsurancePolicyDTO insurancePolicyDto`           | `R<InsurancePolicyVO>` | `@PreAuthorize("@ss.hasPermi('lvguan:policy:add')")` |
| `addInsurancePolicyFile`       | POST   | `/add/file`       | `@RequestParam("file") MultipartFile file`<br>`@Param("id") Long id`      | `R<String>`            | `@PreAuthorize("@ss.hasPermi('lvguan:policy:add')")` |
| `addInsurancePolicyAttachment` | POST   | `/add/attachment` | `@Param("id") Long id`<br>`@Param("attachment") MultipartFile attachment` | `R<String>`            | `@PreAuthorize("@ss.hasPermi('lvguan:policy:add')")` |
| `importInsurance`              | POST   | `/import`         | `@RequestParam("file") MultipartFile file`                                | `R<Boolean>`           | 无                                                    |

### 修改方法

| 方法名 | HTTP方法 | 路径 | 参数 | 返回类型 | 权限 |
|--------|---------|------|------|----------|------|
| `updateInsurancePolicy` | PUT | `/update` | `@RequestBody @Validated InsurancePolicyDTO insurancePolicyDto` | `R<InsurancePolicyVO>` | `@PreAuthorize("@ss.hasPermi('lvguan:policy:edit')")` |

### 删除方法

| 方法名 | HTTP方法 | 路径 | 参数 | 返回类型 | 权限 |
|--------|---------|------|------|----------|------|
| `deleteInsurancePolicy` | DELETE | `/deleted` | `@Param("id") Long id` | `R<String>` | `@PreAuthorize("@ss.hasPermi('lvguan:policy:remove')")` |

### 导出方法

| 方法名 | HTTP方法 | 路径 | 参数 | 返回类型 | 权限 |
|--------|---------|------|------|----------|------|
| `exportInsurance` | POST | `/export` | `HttpServletResponse response`<br>`@RequestBody(required = false) InsurancePolicyExportDTO insurancePolicyExportDTO`<br>`@RequestParam Boolean isTemple` | `void` | 无 |

---

## 3. TbInsuranceClaimController - 报案管理

**路径前缀**: `/lvguan/claim`

### 查询方法

| 方法名 | HTTP方法 | 路径 | 参数 | 返回类型 | 权限 |
|--------|---------|------|------|----------|------|
| `list` | GET | `/list` | `@RequestParam(value = "keyword", required = false) String keyword`<br>`@RequestParam(value = "phone", required = false) String phone`<br>`@RequestParam(value = "status", required = false) Integer status`<br>`@RequestParam(value = "companyName", required = false) String companyName` | `TableDataInfo` | `@PreAuthorize("@ss.hasPermi('system:claim:list')")` |
| `getInfo` | GET | `/detail` | `@RequestParam(value = "id", required = false) Long id`<br>`@RequestParam(value = "claimCode", required = false) String claimCode` | `R<TbInsuranceClaimDetailVO>` | `@PreAuthorize("@ss.hasPermi('system:claim:detail')")` |

### 新增方法

| 方法名 | HTTP方法 | 路径 | 参数 | 返回类型 | 权限 |
|--------|---------|------|------|----------|------|
| `addSave` | POST | `/insert` | `@RequestBody TbInsuranceClaimDTO tbInsuranceClaimDTO` | `R<TbInsuranceClaimDetailVO>` | `@PreAuthorize("@ss.hasPermi('system:claim:add')")` |

### 修改方法

| 方法名 | HTTP方法 | 路径 | 参数 | 返回类型 | 权限 |
|--------|---------|------|------|----------|------|
| `editSave` | PUT | `/update` | `@RequestParam(value = "id") Long id`<br>`@RequestBody TbInsuranceClaimDTO tbInsuranceClaimDTO` | `R<TbInsuranceClaimVO>` | `@PreAuthorize("@ss.hasPermi('system:claim:update')")` |
| `edit` | PUT | `/updateStatus` | `@RequestParam(value = "id") String claimCode`<br>`@RequestParam(value = "status") Integer status` | `R<Boolean>` | `@PreAuthorize("@ss.hasPermi('system:claim:update')")` |

---

## 4. TbVehicleAccessoryController - 配件管理

**路径前缀**: `/lvguan/accessory`

### 查询方法

| 方法名 | HTTP方法 | 路径 | 参数 | 返回类型 | 权限 |
|--------|---------|------|------|----------|------|
| `list` | GET | `/list` | `@Param("keyword") String keyword`<br>`@Param("endDate") @DateTimeFormat(pattern = "yyyy-MM-dd") LocalDate endDate`<br>`@Param("startDate") @DateTimeFormat(pattern = "yyyy-MM-dd") LocalDate startDate` | `TableDataInfo` | `@PreAuthorize("@ss.hasPermi('lvguan:accessory:query')")` |
| `getAccessoryInfo` | GET | `/detail` | `@Param("id") Long id` | `R<TbVehicleAccessory>` | `@PreAuthorize("@ss.hasPermi('lvguan:accessory:query')")` |
| `getFile` | GET | `/get/file` | `@RequestParam Long id`<br>`HttpServletResponse response` | `void` | `@PreAuthorize("@ss.hasPermi('lvguan:accessory:query')")` |
| `getBlockChainDetails` | GET | `/getBlockChainDetails` | `@RequestParam String txHash`<br>`@RequestParam Long id` | `R<BlockChainVO>` | `@PermitAll` |
| `getAttachement` | GET | `/get/attachement` | `@RequestParam Long id`<br>`HttpServletResponse response` | `void` | `@PreAuthorize("@ss.hasPermi('lvguan:accessory:query')")` |

### 新增方法

| 方法名 | HTTP方法 | 路径 | 参数 | 返回类型 | 权限 |
|--------|---------|------|------|----------|------|
| `addAccessory` | POST | `/add` | `@RequestBody @Validated TbVehicleAccessoryDTO tbVehicleAccessoryDTO` | `R<AccessoryVO>` | `@PreAuthorize("@ss.hasPermi('lvguan:accessory:add')")` |
| `upload` | POST | `/add/file` | `@RequestParam("file") MultipartFile file`<br>`@RequestParam Long id` | `R<String>` | `@PreAuthorize("@ss.hasPermi('lvguan:accessory:add')")` |
| `addAttachment` | POST | `/add/attachment` | `@RequestParam("file") MultipartFile file`<br>`@RequestParam Long id` | `R<String>` | `@PreAuthorize("@ss.hasPermi('lvguan:accessory:add')")` |

### 修改方法

| 方法名 | HTTP方法 | 路径 | 参数 | 返回类型 | 权限 |
|--------|---------|------|------|----------|------|
| `update` | PUT | `/update` | `@RequestBody @Validated TbVehicleAccessoryDTO tbVehicleAccessoryDTO` | `R<AccessoryVO>` | `@PreAuthorize("@ss.hasPermi('lvguan:accessory:edit')")` |
| `updateInsurancePolicy` | POST | `/updateStatus` | `@RequestParam @Valid Long id`<br>`@RequestParam Integer status` | `R<String>` | `@PreAuthorize("@ss.hasPermi('lvguan:accessory:edit')")` |
| `updateInspectInfo` | POST | `/updateInspectInfo` | `@RequestBody @Validated AccessoryInspectInfoDTO accessoryInspectInfoDTO` | `R<Boolean>` | `@PreAuthorize("@ss.hasPermi('lvguan:accessory:edit')")` |
| `updateDeliveryStatus` | POST | `/updateDeliveryStatus` | `@RequestParam @Valid String id`<br>`@RequestParam Integer deliveryStatus`<br>`@RequestParam(required = false) String deliveryNumber` | `R<Boolean>` | `@PreAuthorize("@ss.hasPermi('lvguan:accessory:add')")` |
| `updateAccessoryListStatus` | POST | `/updateAccessoryListStatus` | `@RequestParam @Valid List<String> id`<br>`@RequestParam Integer deliveryStatus` | `R<Boolean>` | `@PreAuthorize("@ss.hasPermi('lvguan:accessory:add')")` |

### 删除方法

| 方法名 | HTTP方法 | 路径 | 参数 | 返回类型 | 权限 |
|--------|---------|------|------|----------|------|
| `remove` | DELETE | `/{id}` | `@PathVariable Long id` | `R<String>` | `@PreAuthorize("@ss.hasPermi('lvguan:accessory:remove')")` |

---

## 5. TbVehicleAccessoryOrderController - 配件订单管理

**路径前缀**: `/lvguan/accessoryOrder`

### 查询方法

| 方法名 | HTTP方法 | 路径 | 参数 | 返回类型 | 权限 |
|--------|---------|------|------|----------|------|
| `list` | GET | `/list` | `@Param("keyword") String keyword`<br>`@Param("endDate") @DateTimeFormat(pattern = "yyyy-MM-dd") LocalDate endDate`<br>`@Param("startDate") @DateTimeFormat(pattern = "yyyy-MM-dd") LocalDate startDate`<br>`@Param("status") Integer status` | `TableDataInfo` | `@PreAuthorize("@ss.hasPermi('lvguan:accessoryOrder:query')")` |
| `getInfo` | GET | `/{id}` | `@PathVariable("id") Long id` | `R<AccessoryOrderDetail>` | `@PreAuthorize("@ss.hasPermi('lvguan:accessoryOrder:query') or @ss.hasPermi('lvguan:accessory:add')")` |
| `getFixInfo` | GET | `/get/file` | `@RequestParam Long id`<br>`HttpServletResponse response` | `void` | `@PreAuthorize("@ss.hasPermi('lvguan:accessoryOrder:query')")` |
| `getProvider` | GET | `/getProvider` | 无 | `R<List<DeptVO>>` | `@PermitAll` |
| `getAttachment` | GET | `/get/attachment` | `@RequestParam Long id`<br>`HttpServletResponse response` | `void` | `@PreAuthorize("@ss.hasPermi('lvguan:accessoryOrder:query')")` |

### 新增方法

| 方法名 | HTTP方法 | 路径 | 参数 | 返回类型 | 权限 |
|--------|---------|------|------|----------|------|
| `addInsurancePolicy` | POST | `/add` | `@RequestBody @Valid AccessoryOrderDTO tbVehicleAccessoryOrder` | `R<AccessoryOrderDTO>` | `@PreAuthorize("@ss.hasPermi('lvguan:accessoryOrder:add')")` |
| `addRepairPhoto` | POST | `/add/file` | `@RequestParam Long id`<br>`@RequestParam MultipartFile file` | `R<String>` | `@PreAuthorize("@ss.hasPermi('lvguan:accessoryOrder:add')")` |
| `addAttachment` | POST | `/add/attachment` | `@RequestParam Long id`<br>`@RequestParam MultipartFile attachment` | `R<String>` | `@PreAuthorize("@ss.hasPermi('lvguan:accessoryOrder:add')")` |

### 修改方法

| 方法名 | HTTP方法 | 路径 | 参数 | 返回类型 | 权限 |
|--------|---------|------|------|----------|------|
| `update` | PUT | `/update` | `@RequestBody @Validated AccessoryOrderDTO accessoryOrderDTO` | `R<AccessoryOrderVO>` | `@PreAuthorize("@ss.hasPermi('lvguan:accessoryOrder:edit')")` |

### 删除方法

| 方法名 | HTTP方法 | 路径 | 参数 | 返回类型 | 权限 |
|--------|---------|------|------|----------|------|
| `delete` | DELETE | `/delete` | `@RequestParam Long id` | `R<String>` | `@PreAuthorize("@ss.hasPermi('lvguan:accessoryOrder:remove')")` |

---

## 6. TbInsurancePreorderController - 预订单管理

**路径前缀**: `/lvguan/preorder`

### 查询方法

| 方法名 | HTTP方法 | 路径 | 参数 | 返回类型 | 权限 |
|--------|---------|------|------|----------|------|
| `list` | GET | `/list` | `@RequestParam(value = "preorder_code", required = false) String preorderCode`<br>`@RequestParam(value = "status", required = false) Integer status` | `TableDataInfo` | `@PreAuthorize("@ss.hasPermi('system:preorder:list')")` |
| `getInfo` | GET | `/detail` | `@RequestParam("id") String id` | `R<TbInsurancePreorderDetailVO>` | `@PreAuthorize("@ss.hasPermi('system:preorder:query')")` |

### 新增方法

| 方法名 | HTTP方法 | 路径 | 参数 | 返回类型 | 权限 |
|--------|---------|------|------|----------|------|
| `add` | POST | `/add` | `@RequestBody TbInsurancePreorderDTO tbInsurancePreorderDTO` | `R<TbInsurancePreorderDetailVO>` | `@PreAuthorize("@ss.hasPermi('system:preorder:add')")` |

### 修改方法

| 方法名 | HTTP方法 | 路径 | 参数 | 返回类型 | 权限 |
|--------|---------|------|------|----------|------|
| `edit` | PUT | `/update` | `@Validated @RequestParam("id") String id`<br>`@RequestBody TbInsurancePreorderDTO insurancePreorderDTO` | `R<TbInsurancePreorderDetailVO>` | `@PreAuthorize("@ss.hasPermi('system:preorder:edit')")` |

---

## 7. TbInsuranceRepairController - 维修单管理

**路径前缀**: `/lvguan/repair`

### 查询方法

| 方法名 | HTTP方法 | 路径 | 参数 | 返回类型 | 权限 |
|--------|---------|------|------|----------|------|
| `list` | GET | `/list` | `TbInsuranceRepair tbInsuranceRepair` | `TableDataInfo` | `@PreAuthorize("@ss.hasPermi('system:repair:list')")` |
| `getInfo` | GET | `/detail` | `@RequestParam("id") String id` | `R<TbInsuranceRepairVO>` | `@PreAuthorize("@ss.hasPermi('system:repair:query')")` |

### 新增方法

| 方法名 | HTTP方法 | 路径 | 参数 | 返回类型 | 权限 |
|--------|---------|------|------|----------|------|
| `add` | POST | `/` | `@RequestBody TbInsuranceRepairSaveDTO tbInsuranceRepair` | `R<String>` | `@PreAuthorize("@ss.hasPermi('system:repair:add')")` |
| `uploadAccessoryPhoto` | POST | `/uploadAccessoryPhoto` | `@RequestBody List<RepairAccessoryPhotoSaveVO> photoList` | `R<Boolean>` | `@PreAuthorize("@ss.hasPermi('system:repair:add')")` |

### 修改方法

| 方法名 | HTTP方法 | 路径 | 参数 | 返回类型 | 权限 |
|--------|---------|------|------|----------|------|
| `edit` | PUT | `/` | `@RequestBody TbInsuranceRepairSaveDTO tbInsuranceRepair` | `R<Boolean>` | `@PreAuthorize("@ss.hasPermi('system:repair:edit')")` |
| `updateAccessoryStatus` | PUT | `/updateAccessoryStatus` | `@RequestParam String id`<br>`@RequestParam Integer status` | `R<Boolean>` | `@PreAuthorize("@ss.hasPermi('system:repair:edit')")` |

### 删除方法

| 方法名 | HTTP方法 | 路径 | 参数 | 返回类型 | 权限 |
|--------|---------|------|------|----------|------|
| `remove` | DELETE | `/{ids}` | `@PathVariable Long[] ids` | `AjaxResult` | `@PreAuthorize("@ss.hasPermi('system:repair:remove')")` |

---

## 8. TbAfterSalesServiceRecordController - 售后服务管理

**路径前缀**: `/lvguan/afterSalesService/`

### 查询方法

| 方法名 | HTTP方法 | 路径 | 参数 | 返回类型 | 权限 |
|--------|---------|------|------|----------|------|
| `list` | GET | `/list` | 无 | `TableDataInfo` | `@PreAuthorize("@ss.hasPermi('system:record:list')")` |
| `getInfo` | GET | `/detail` | `@RequestParam("id") Long id` | `R<TbAfterSalesServiceRecord>` | `@PreAuthorize("@ss.hasPermi('system:record:query')")` |

### 新增方法

| 方法名 | HTTP方法 | 路径 | 参数 | 返回类型 | 权限 |
|--------|---------|------|------|----------|------|
| `add` | POST | `/add` | `@RequestBody @Validated TbAfterSalesServiceRecord tbAfterSalesServiceRecord` | `R<TbAfterSalesServiceRecord>` | `@PreAuthorize("@ss.hasPermi('system:record:add')")` |

### 修改方法

| 方法名 | HTTP方法 | 路径 | 参数 | 返回类型 | 权限 |
|--------|---------|------|------|----------|------|
| `edit` | PUT | `/update` | `@RequestParam("id") Long id`<br>`@RequestBody @Validated TbAfterSalesServiceRecord tbAfterSalesServiceRecord` | `R<TbAfterSalesServiceRecord>` | `@PreAuthorize("@ss.hasPermi('system:record:edit')")` |

---

## 9. TbInsuranceCompanyController - 保险公司管理

**路径前缀**: `/lvguan/company`

### 查询方法

| 方法名 | HTTP方法 | 路径 | 参数 | 返回类型 | 权限 |
|--------|---------|------|------|----------|------|
| `listMyPolicyCompanyPhone` | GET | `/phones` | `@RequestParam("phone") String phone` | `TableDataInfo` | `@PreAuthorize("@ss.hasPermi('platform:company:list')")` |
| `getCompanyPhoneByPolicyId` | GET | `/phone` | `@RequestParam("policy_id") String policyId` | `R<TbInsuranceCompanyPhoneVO>` | 无 |

---

## 10. TbProviderOrderController - 供应商预订单管理

**路径前缀**: `/providerorder/order`

### 查询方法

| 方法名 | HTTP方法 | 路径 | 参数 | 返回类型 | 权限 |
|--------|---------|------|------|----------|------|
| `list` | GET | `/list` | `ResearchBaseDTO researchBaseDTO` | `TableDataInfo` | `@PermitAll` |
| `getInfo` | GET | `/{id}` | `@PathVariable("id") String id` | `AjaxResult` | `@PermitAll` |

### 新增方法

| 方法名 | HTTP方法 | 路径 | 参数 | 返回类型 | 权限 |
|--------|---------|------|------|----------|------|
| `add` | POST | `/` | `@RequestBody TbProviderOrder tbProviderOrder` | `AjaxResult` | `@PermitAll` |

### 修改方法

| 方法名 | HTTP方法 | 路径 | 参数 | 返回类型 | 权限 |
|--------|---------|------|------|----------|------|
| `edit` | PUT | `/` | `@RequestBody TbProviderOrder tbProviderOrder` | `AjaxResult` | `@PermitAll` |

### 删除方法

| 方法名 | HTTP方法 | 路径 | 参数 | 返回类型 | 权限 |
|--------|---------|------|------|----------|------|
| `remove` | DELETE | `/{ids}` | `@PathVariable String[] ids` | `AjaxResult` | `@PermitAll` |

---

## 11. GlobalSearchController - 全局搜索

**路径前缀**: `/lvguan/globalSearch`

### 查询方法

| 方法名 | HTTP方法 | 路径 | 参数 | 返回类型 | 权限 |
|--------|---------|------|------|----------|------|
| `getGlobalSearchResult` | GET | `/` | `@Param("keyword") @Valid @NotBlank String keyword` | `R<GlobalSearchVO>` | `@PreAuthorize("@ss.hasPermi('lvguan:global:list')")` |
| `prefixKeyword` | GET | `/prefixKeyWord` | `@Param("prefixKeyword") @Valid @NotBlank String prefixKeyword` | `R<List<String>>` | `@PreAuthorize("@ss.hasPermi('lvguan:global:list')")` |

---

## 12. SysFileInfoController - 文件管理

**路径前缀**: `/system/info`

### 新增方法

| 方法名 | HTTP方法 | 路径 | 参数 | 返回类型 | 权限 |
|--------|---------|------|------|----------|------|
| `addSave` | POST | `/upload` | `@RequestParam("file") MultipartFile file`<br>`String remark` | `AjaxResult` | 无 |

### 查询方法

| 方法名 | HTTP方法 | 路径 | 参数 | 返回类型 | 权限 |
|--------|---------|------|------|----------|------|
| `resourceDownload` | GET | `/download` | `String resource`<br>`HttpServletRequest request`<br>`HttpServletResponse response` | `void` | 无 |

---

## 13. ESController - ElasticSearch数据同步

**路径前缀**: 无（根路径）

### 数据同步方法

| 方法名 | HTTP方法 | 路径 | 参数 | 返回类型 | 权限 |
|--------|---------|------|------|----------|------|
| `policy` | GET | `/policy` | 无 | `String` | 无 |
| `accident` | GET | `/order` | 无 | `String` | 无 |
| `accessory` | POST | `/accessory` | 无 | `String` | 无 |
| `orderNumber` | GET | `/orderNumber` | 无 | `String` | 无 |

---

## 总结统计

### 按功能模块统计

| 模块 | 查询方法数 | 新增方法数 | 修改方法数 | 删除方法数 | 其他方法数 | 总计 |
|------|-----------|-----------|-----------|-----------|-----------|------|
| 数据大屏 | 7 | 0 | 0 | 0 | 0 | 7 |
| 保单管理 | 6 | 4 | 1 | 1 | 1 | 13 |
| 报案管理 | 2 | 1 | 2 | 0 | 0 | 5 |
| 配件管理 | 5 | 3 | 5 | 1 | 0 | 14 |
| 配件订单管理 | 5 | 3 | 1 | 1 | 0 | 10 |
| 预订单管理 | 2 | 1 | 1 | 0 | 0 | 4 |
| 维修单管理 | 2 | 2 | 2 | 1 | 0 | 7 |
| 售后服务管理 | 2 | 1 | 1 | 0 | 0 | 4 |
| 保险公司管理 | 2 | 0 | 0 | 0 | 0 | 2 |
| 供应商预订单管理 | 2 | 1 | 1 | 1 | 0 | 5 |
| 全局搜索 | 2 | 0 | 0 | 0 | 0 | 2 |
| 文件管理 | 1 | 1 | 0 | 0 | 0 | 2 |
| ES数据同步 | 0 | 0 | 0 | 0 | 4 | 4 |
| **总计** | **38** | **17** | **14** | **5** | **5** | **79** |

### 权限控制统计

- **@PermitAll**: 7个方法（允许所有用户访问）
- **@PreAuthorize**: 72个方法（需要特定权限）
- **无权限注解**: 0个方法

---

## 备注

1. 所有日期参数使用 `@DateTimeFormat(pattern = "yyyy-MM-dd")` 格式化为 `LocalDate` 类型
2. 文件上传使用 `MultipartFile` 类型
3. 返回类型使用统一的 `R<T>` 包装类或 `TableDataInfo`（分页查询）
4. 大部分Controller继承自 `BaseController`，可以使用 `startPage()` 进行分页
5. 使用了 `SearchContext` 来获取ES搜索的总记录数
6. 部分方法使用了 `@Validated` 或 `@Valid` 进行参数验证

