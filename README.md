# es-gitops-demo-src

Here are the yamls that I use for deploying my own events test environment inside IBM on Fyre, which consists of Event Streams, Event Processing, and Flink.

The Argo App definitions are in root directory.

I use ArgoCD (Openshift Gitops) for this, and point Argo at a repo on github.ibm.com.

RE: your email about KafkaTopics, I don't personally do topics via argo (I used to, but I've since started packaging my topics up with my events applications to have one self-contained deployable unit "thing"), but I'm sure they'll work just fine. I've just added the file topics.yaml into the repo as an example. The gotchas is to make sure that you have 
```yaml
spec:
  strimziOverrides:
    entityOperator:
      topicOperator: {}
```
in your eventStreams yaml.

I do most stuff with argo syncwaves, approximately in this order:

- Cert-manager
- Operators
- Actual CRs
- Difficult Stuff
- PipelineRuns (I build my flink custom container images inside a tekton pipeline)

Things to note:
- The pipelineRun stuff isn't perfect, and gets stuck a lot. I often have to re-sync argo to give it a kick, I think I'm being too clever with postsync hooks. This really ought to be reworked
- I deploy all my operators with custom catalogSources generated via the CASE tool to lock them to particular versions and to allow for a very gitopsy approach to operator versions. I am starting to think that this is more effort than it is worth, YMMV
- Lots of my CRs have `version: latest` in their yamls - this is obviously fine for a quick and dirty internal environment but you'd clean this up for a proper customer deploy
