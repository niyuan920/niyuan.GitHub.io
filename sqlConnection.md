.NET CORE MVC中访问数据库

首先确保都安装CLI相关工具引用，在Package Manager console中运行以下命令：

    Install-Package Microsoft.EntityFrameworkCore.Design
    Install-Package Microsoft.EntityFrameworkCore.Tools
    
需要使用 Scaffold-DbContext 命令生成数据实体类型，数据表必须有主键！

    Scaffold-DbContext "Server=.;Database=IDEA;Trusted_Connection=True;uid=sa;pwd=1005" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models -Context SaleContext -DataAnnotations -Force

EF Core查询方法参考
https://github.com/imwyw/.net/blob/master/CrossPlatform/EFCore.md

手写sql查询函数

        private static DbCommand CreateCommand(DatabaseFacade facade, string sql, out DbConnection connection, params object[] parameters)
        {
            var conn = facade.GetDbConnection();
            connection = conn;
            conn.Open();
            var cmd = conn.CreateCommand();
            if (facade.IsSqlServer())
            {
                cmd.CommandText = sql;
                cmd.Parameters.AddRange(parameters);
            }
            return cmd;
        }

        public static DataTable SqlQuery(this DatabaseFacade facade, string sql, params object[] parameters)
        {
            var command = CreateCommand(facade, sql, out DbConnection conn, parameters);
            var reader = command.ExecuteReader();
            var dt = new DataTable();
            dt.Load(reader);
            reader.Close();
            conn.Close();
            return dt;
        }

        public static List<T> SqlQuery<T>(this DatabaseFacade facade, string sql, params object[] parameters) where T : class, new()
        {
            var dt = SqlQuery(facade, sql, parameters);
            return dt.ToList<T>();
        }

        public static List<T> ToList<T>(this DataTable dt) where T : class, new()
        {
            var propertyInfos = typeof(T).GetProperties();
            var list = new List<T>();
            foreach (DataRow row in dt.Rows)
            {
                var t = new T();
                foreach (PropertyInfo p in propertyInfos)
                {
                    if (dt.Columns.IndexOf(p.Name) != -1 && row[p.Name] != DBNull.Value)
                        p.SetValue(t, row[p.Name], null);
                }
                list.Add(t);
            }
            return list;
        }
    }
.NET MVC中访问数据库

    public class EFUtility
    {
        /// <summary>
        /// EF 查询 DataTable 
        /// </summary>
        /// <param name="sql"></param>
        /// <param name="sqlParams"></param>
        /// <returns></returns>
        public static DataTable GetDataTableBySql(string sql, SqlParameter[] sqlParams)
        {
            using (LearnContext db=new LearnContext())
            {
                IDbCommand cmd = db.Database.Connection.CreateCommand();
                cmd.CommandText = sql;
                if (sqlParams != null && sqlParams.Length > 0)
                {
                    foreach (var item in sqlParams)
                    {
                        cmd.Parameters.Add(item);
                    }
                }

                SqlDataAdapter adapter = new SqlDataAdapter();
                adapter.SelectCommand = cmd as SqlCommand;

                DataTable dt = new DataTable();
                adapter.Fill(dt);

                return dt;
            }
        }

        /// <summary>
        /// 查询统计数目
        /// </summary>
        /// <param name="sql"></param>
        /// <param name="parameters"></param>
        /// <returns></returns>
        public static int GetCount(string sql, IEnumerable<SqlParameter> parameters)
        {
            // clone 为解决 【另一个 SqlParameterCollection 中已包含 SqlParameter。】问题
            var paramClone = parameters.Select(t => ((ICloneable)t).Clone());
            string sqlCount = ConvertSqlCount(sql);

            using (LearnContext db=new LearnContext())
            {
                int count = db.Database.SqlQuery<int>(sqlCount, paramClone.ToArray())
                    .First();
                return count;
            }
        }

        /// <summary>
        /// 执行查询
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="sql"></param>
        /// <param name="parameters"></param>
        /// <returns></returns>
        public static List<T> GetList<T>(string sql, IEnumerable<SqlParameter> parameters)
        {
            // clone 为解决 【另一个 SqlParameterCollection 中已包含 SqlParameter。】问题
            var paramClone = parameters.Select(t => ((ICloneable)t).Clone());

            using (LearnContext db = new LearnContext())
            {
                var query = db.Database
                    .SqlQuery<T>(sql, paramClone.ToArray())
                    .AsQueryable();

                List<T> list = query.ToList();

                return list;
            }
        }

        /// 处理sql，将 select a,b,c from xxx 转换为 select count(1) from xxx结构
        /// 快速统计数目
        /// </summary>
        /// <param name="sql"></param>
        /// <returns></returns>
        private static string ConvertSqlCount(string sql)
        {
            /* 正则替换，忽略大小写
            \s空白符，\S非空白符，[\s\S]是任意字符
            */
            Regex reg = new Regex(@"select[\s\S]*from", RegexOptions.IgnoreCase);
            string sqlCount = reg.Replace(sql, "SELECT COUNT(1) FROM ");
            return sqlCount;
        }
    }
三层架构中访问数据库

    class MysqlHelper
    {
        static readonly string CONNSTR = "server=.;database=companysales;uid=yuan;pwd=88888888;";
        public static int ExecuteMyDML(string sql, SqlParameter[] sqlParams)
        {
            SqlConnection conn = new SqlConnection(CONNSTR);
            try
            {
                conn.Open();
                SqlCommand cmd = new SqlCommand();
                cmd.Connection = conn;

                cmd.CommandText = sql;
                if (null != sqlParams && sqlParams.Length > 0)
                {
                    cmd.Parameters.AddRange(sqlParams);
                }
                int res = cmd.ExecuteNonQuery();
                return res;
            }
            catch (Exception ex)
            {
                Console.WriteLine(ex.Message);
                return -1;
            }
            finally
            {
                if (conn.State != System.Data.ConnectionState.Closed)
                {
                    conn.Close();
                }
            }
        }
        public static DataTable QueryData(string sql)
        {
            SqlConnection conn = new SqlConnection(CONNSTR);

            try
            {
                SqlCommand cmd = new SqlCommand();
                cmd.Connection = conn;

                cmd.CommandText = sql;
                SqlDataAdapter adapter = new SqlDataAdapter(cmd);
                DataTable dt = new DataTable();
                adapter.Fill(dt);

                return dt;
            }
            catch (Exception ex)
            {
                Console.WriteLine(ex.Message);
                return null;
            }
            finally
            {
                if (conn.State != System.Data.ConnectionState.Closed)
                {
                    conn.Close();
                }
            }
        
            }
      public  static object ExecuteMyObject(string sql, SqlParameter[] sqlParams)
        {
            SqlConnection conn = new SqlConnection(CONNSTR);
            try
            {
                conn.Open();

                SqlCommand cmd = new SqlCommand();
                cmd.Connection = conn;

                cmd.CommandText = sql;

                //cmd.Parameters.AddWithValue("@UID", uid);
                //cmd.Parameters.AddWithValue("@PWD", pwd);
                if (null != sqlParams && sqlParams.Length > 0)
                {
                    cmd.Parameters.AddRange(sqlParams);
                }

                object res = cmd.ExecuteScalar();
                return res;
            }
            catch (Exception ex)
            {
                Console.WriteLine(ex.Message);
                return null;
            }
            finally
            {
                if (conn.State != System.Data.ConnectionState.Closed)
                {
                    conn.Close();
                }
            }
        }
    }


