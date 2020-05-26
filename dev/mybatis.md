## 注解模式

```java
@Mapper
@Repository
public interface ModelMapper {

    @Insert("insert into ACT_DE_MODEL (id, name) " +
            "values (#{id, jdbcType=VARCHAR}, #{name, jdbcType=VARCHAR}, #{key, jdbcType=VARCHAR})")
    void insert(AbstractModel model);

    @UpdateProvider(type = ModelMapperProvider.class, method = "updateById")
    void update(AbstractModel model);

    @Select("select * from ACT_DE_MODEL where id = #{id, jdbcType=VARCHAR}")
    @Results(id = "modelResult", value = {
            @Result(column = "id", property = "id"),
            @Result(column = "name", property = "name"),
            @Result(column = "model_key", property = "key"),
    })
    AbstractModel findById(@Param("id") String id);

    @Select("select * from ACT_DE_MODEL where model_key = #{key, jdbcType=VARCHAR} " +
            "and model_type = #{modelType, jdbcType=INTEGER} " +
            "order by version desc " +
            "limit 1")
    @ResultMap("modelResult")
    List<AbstractModel> findByKeyAndType(@Param("key") String key, @Param("modelType") Integer modelType);

    class ModelMapperProvider {
        public String updateById(AbstractModel model) {
            return new SQL() {{
                UPDATE("act_de_model");
                if (model.getName() != null)
                    SET("name = #{name, jdbcType=VARCHAR}");
                if (model.getKey() != null)
                    SET("model_key = #{key, jdbcType=VARCHAR}");
                WHERE("id = #{id, jdbcType=VARCHAR}");
            }}.toString();
        }
    }
}

```

- @Mapper
- 