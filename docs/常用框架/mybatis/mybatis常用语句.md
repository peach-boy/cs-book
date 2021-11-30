# mybatis使用经验

#### in关键字
```
  <if test="cardSampleIdList != null and cardSampleIdList.size()>0">
        and cardSampleId in
            <foreach item="item" index="index" collection="cardSampleIdList" open="(" separator="," close=")">
                #{item}
            </foreach>
        </if>
```

#### update 
```
  update ExchangePointRecord
        <trim prefix="set" suffixOverrides=",">
            <if test="status != null">
                status = #{status},
            </if>
            <if test="refundOrderId != null">
                refundOrderId = #{refundOrderId},
            </if>
            <if test="enterpriseTransferId != null">
                enterpriseTransferId = #{enterpriseTransferId},
            </if>
            <if test="personAssignBillId != null">
                personAssignBillId = #{personAssignBillId},
            </if>
        </trim>
        where id = #{id}
```

#### insert(返回Id)

* mapper
```<insert id="add" keyProperty="id" useGeneratedKeys="true" parameterType="ExchangePointRecordDO">
        insert into ExchangePointRecord (<include refid="insert_column"/>)
        values
        (
        #{cardCode,jdbcType=VARCHAR},
        #{memberId,jdbcType=INTEGER},
        #{exchangeAmount,jdbcType=VARCHAR},
        #{status,jdbcType=TINYINT}
        )
    </insert>
```
* dao
```
exchangePointRecordDao.add(exchangePointRecordDO);
//此处exchangePointRecordDO中id已经自动设为插入后的Id
exchangePointRecordDO.getId();
```

#### batchInsert
```
 <insert id="batchInsert" parameterType="java.util.List" useGeneratedKeys="true" keyProperty="id">
        insert into RefundRecycleDetail (<include refid="insert_column"/>)
        values
        <foreach collection="list" item="detail" open="" separator="," close="">
            (
            #{detail.refundRecycleId},
            #{detail.cardOutStorageId},
            #{detail.cardCode},
            #{detail.cardFactoryId},
            #{detail.cardSampleId},
            #{detail.cardStatus},
            #{detail.status},
            #{detail.descript}
            )
        </foreach>
    </insert>
```