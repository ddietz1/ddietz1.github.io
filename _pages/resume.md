---
layout: page
title: Resume
permalink: /resume/
---

<style>
  /* Hide page header */
  .page-header {
    display: none;
  }
  
  /* Make page container full-width */
  .page-container {
    max-width: 100%;
    width: 100%;
    padding: 0;
    margin: 0;
  }
  
  /* Make page content full-width */
  .page-content {
    margin: 0;
    /* Added top padding to prevent the resume content from touching the header */
    padding-top: 0.5rem;
  }
  
  /* Full-screen PDF viewer */
  .resume-container {
    width: 50vw;
    height: calc(100vh - 80px); /* Account for navigation */
   
    margin-left: auto;
    margin-right: auto;
  }
  
  .pdf-embed {
    width: 100%;
    height: 100%;
    border: none;
    display: block;
  }
  
  /* Mobile optimization */
  @media (max-width: 768px) {
    .resume-container {
      height: calc(100vh - 60px);
      padding-top: 1rem; /* Adjust padding for smaller header height on mobile */
    }
  }
</style>

<div class="resume-container">
  <embed 
    src="{{ '/assets/resume/Resume.pdf' | relative_url }}" 
    type="application/pdf" 
    class="pdf-embed"
    title="Resume"
  />
  
  <noscript>
    <p style="padding: 2rem; text-align: center;">
      Your browser does not support embedded PDFs. 
      <a href="{{ '/assets/resume/resume.pdf' | relative_url }}" download>Download the resume</a> instead.
    </p>
  </noscript>
</div>
