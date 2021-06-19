.NET CORE MVC中访问数据库

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


