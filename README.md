<h1>Passo a passo para rodar o projeto:</h1>
<div>
    <h2>Passo 1: Configurando o ambiente</h2>
    <ol>
        <li><p>Certifique-se de ter o Docker instalado em sua máquina.</p></li>
        <li>
            <p>Verifique se as portas necessárias para o projeto estão disponíveis. Se necessário, modifique-as.</p>
            <p style="margin-left: 20px">As portas utilizadas são:</p>
            <ul>
                <li>9104 : mysql-exporter</li>
                <li>6379 : redis</li>
                <li>8080 : wordpress</li>
                <li>9090 : prometheus</li>
                <li>3000 : grafana</li>
                <li>8081 : cadivisor</li>
            </ul>
        </li>
    </ol>
</div>
<hr/>
<div>
    <h2>Passo 2: Clonar e acessar repositório do projeto</h2>
    <ol>
        <li>
            <p>Escolha uma pasta no seu computador que não contenha caracteres especiais.</p>
            <p>Abra o Git Bash e execute o comando a seguir:</p>
            <pre><code>git clone https://github.com/mafebordignon/ProjetoDocker.git</code></pre>
        </li>
        <li>
            <p>Acesse a pasta do projeto:</p>
            <pre><code>cd ProjetoDocker</code></pre>
        </li>
    </ol>
</div>
<hr/>
<div>
    <h2>Passo 3: Executar o projeto</h2>
    <ol>
        <li>
            <p>Inicie o Docker Swarm para rodar o projeto:</p>
            <pre><code>docker swarm init</code></pre>
        </li>
        <li>
            <p>Execute o seguinte comando:</p>
            <pre><code>docker stack deploy -c docker-compose.yml "nome_stack"</code></pre>
            <p>Isso iniciará os serviços definidos no arquivo <code>.yml</code>, que são:</p>
            <ul>
                <li>mysql - banco de dados</li>
                <li>mysql-exporter - exporta dados do MySQL para o Prometheus</li>
                <li>redis - usado para armazenar em cache as consultas do WordPress</li>
                <li>wordpress - nosso site</li>
                <li>prometheus - ferramenta de monitoramento</li>
                <li>grafana - visualiza os dados do Prometheus em dashboards</li>
                <li>cadvisor - monitora o Docker e seus containers</li>
            </ul>
        </li>
    </ol>
</div>
<div>
    <h2>Passo 4: Configurar o WordPress</h2>
    <ol>
        <li>Acesse o link: <code>http://localhost:8080</code> no seu navegador.</li>
        <li>
            <p>Siga as instruções para configurar o WordPress. Não se preocupe com os dados inseridos, são apenas para teste.</p>
        </li>
        <li>
            <p>Após a configuração, você será redirecionado para a página de administração:</p>
            <img src="./md/image_wp_admin.png"/>                  
        </li>
        <li>
            <p>Acesse a seção de plugins, onde estão os plugins instalados:</p>
            <img src="./md/image_wp_plugins.png" />
        </li>
        <li>
            <p>Clique em "Adicionar plugin" para ir à página de adição de plugins:</p>
            <img src="./md/image_wp_adicionar_plugin.png"/>
        </li>
        <li>
            <p>Pesquise pelo plugin <code>Redis Object Cache</code>:</p>
            <img src="./md/image_wp_redis.png"/>
            <p>Instale e ative o plugin. Você verá a seguinte mensagem de erro de conexão:</p>
            <img src="./md/image_wp_erro_redis.png"/>
        </li>
    </ol>
</div>
<div>
    <h2>Corrigir erro de conexão entre o Redis e o Docker</h2>
    <ol>
        <li>
            <p>Digite <code>sudo su</code> no terminal para se tornar o usuário root. Faça login com sua senha de root e execute:</p>
            <pre><code>docker ps</code></pre>
            <p>Encontre o container do WordPress e copie seu <code>CONTAINER ID</code>.</p>
        </li>
        <li>
            <p>Acesse o container com:</p>
            <pre><code>docker exec -it "ID DO SEU CONTAINER" bash</code></pre>
        </li>
        <li>
            <p>Execute os comandos:</p>
            <pre><code>apt update</code></pre>
            <pre><code>apt install nano</code></pre>
            <p>Isso atualizará o apt e instalará o editor nano para editar o arquivo <code>wp-config.php</code>.</p>
        </li>
        <li>
            <p>Edite o arquivo <code>wp-config.php</code> com:</p>
            <pre><code>nano wp-config.php</code></pre>
        </li>
        <li>
            <p>Adicione as seguintes linhas antes de <code>/* That's all, stop editing! Happy publishing. */</code>:</p>
            <pre><code>define('WP_CACHE', true);<br>define('WP_REDIS_HOST', 'redis');<br>define('WP_REDIS_PORT', 6379);</code></pre>
        </li>
        <li>
            <p>Salve o arquivo com <code>CTRL+O</code> e saia com <code>CTRL+X</code>.</p>
        </li>
        <li>
            <p>Atualize a página de plugins do WordPress e veja se o Redis aparece como acessível:</p>
            <img src="./md/image_wp_redis_acessivel.png"/>
            <p>Ative o cache de objeto e o Redis estará configurado.</p>
        </li>
    </ol>
</div>
<div>
    <h2>Passo 5: Acessar o Prometheus</h2>
    <ol>
        <li>
            <p>Acesse <code>http://localhost:9090</code>. Na tela inicial do Prometheus (Graph), você pode executar consultas de métricas.</p>
        </li>
        <li>
            <p>Verifique métricas dos containers:</p>
            <pre>container_cpu_system_seconds_total<br>container_fs_reads_total<br>container_fs_limit_bytes</pre>
            <p>Verifique métricas do MySQL:</p>
            <pre>mysql_exporter_collector_success<br>mysql_global_status_connections<br>mysql_global_status_max_used_connections</pre>
        </li>
    </ol>
</div>
<div>
    <h2>Passo 6: Dashboard do Grafana</h2>
    <ol>
        <li>
            <p>Acesse <code>http://localhost:3000</code>.</p>
        </li>
        <li>
            <p>Entre com o usuário <code>admin</code> e senha <code>admin</code>:</p>
            <img src="./md/image_grafana_login.png"/>
        </li>
        <li>
            <p>Na dashboard, vá em "Connections" e selecione "Data sources":</p>
            <img src="./md/image_grafana_datasource.png"/>
        </li>
        <li>
            <p>Clique em "Add data source" e selecione Prometheus.</p>
        </li>
        <li>
            <p>Configure o Prometheus com a URL <code>http://prometheus:9090</code>:</p>
            <img src="./md/image_grafana_prometheus.png"/>
        </li>
        <li>
            <p>Salve e teste a configuração. Em seguida, crie uma nova dashboard e adicione uma visualização.</p>
        </li>
        <li>
            <p>Selecione o Prometheus no modal e configure a consulta. Execute a consulta e aplique as configurações para gerar o gráfico:</p>
            <img src="./md/image_grafana_setqueries.png"/>
        </li>
    </ol>
</div>
<div>
    <h2>Passo Bônus: Cadvisor</h2>
    <ol>
        <li>
            <p>Para visualizar gráficos dos containers e do Docker, acesse <code>http://localhost:8081</code>.</p>
        </li>
    </ol>
</div>
</body>
