<script>
    export let projectTitle = "BIOM5403 - Visualization Project";
    export let studentInfo = "Darwin Jull, Student No: 101145679";
    export let githubLink = "https://github.com/darwinkjull/vis-5403";
  </script>
  
  <header>
    <h1>{projectTitle}</h1>
    <p>{studentInfo}</p>
    <a href={githubLink} target="_blank">GitHub Repository</a>
  </header>
  
  <style>
    header {
      background-color: #2c3e50;
      color: white;
      padding: 20px;
      text-align: center;
    }
  
    h1 {
      margin: 0;
      font-size: 24px;
    }
  
    p {
      margin: 5px 0;
      font-size: 14px;
    }
  
    a {
      color: #1abc9c;
      text-decoration: none;
      font-size: 14px;
    }
  
    a:hover {
      text-decoration: underline;
    }
  </style>
  